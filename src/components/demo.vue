<template>
    <div id="container"
        :style="'width: '+innerWidth+'px;height: '+innerHeight+'px;'"></div>
</template>
<script>
import * as THREE from 'three';

import Stats from 'three/examples/jsm/libs/stats.module.js';
import { GUI } from 'three/examples/jsm/libs/lil-gui.module.min.js';

import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls.js';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js';
import { EffectComposer } from 'three/examples/jsm/postprocessing/EffectComposer.js';
import { RenderPass } from 'three/examples/jsm/postprocessing/RenderPass.js';
import { UnrealBloomPass } from 'three/examples/jsm/postprocessing/UnrealBloomPass.js';


export default {
    data () {
        return {
            innerWidth: 800,
            innerHeight: 800,
            camera: null,
            stats: null,
            composer: null,
            renderer: null,
            mixer: null,
            gltfChildFour:null,
            plant:null,
            params: {
				exposure: 1.6,
				bloomStrength: 1.5,
				bloomThreshold: 0,
				bloomRadius: 0
			}
        }
    },
    methods: {
        init () {
            var that = this;

			const container = document.getElementById( 'container' );

			this.stats = new Stats();
			container.appendChild( this.stats.dom );

            // 创建渲染器
			this.renderer = new THREE.WebGLRenderer( { antialias: true } );
			this.renderer.setPixelRatio( window.devicePixelRatio );
			this.renderer.setSize( window.innerWidth, window.innerHeight );
			this.renderer.outputColorSpace = THREE.LinearSRGBColorSpace;
			this.renderer.toneMapping = THREE.ReinhardToneMapping;
			container.appendChild( this.renderer.domElement );

            // 创建场景
			const scene = new THREE.Scene();

            // 创建投影相机
			this.camera = new THREE.PerspectiveCamera(45, that.innerWidth / that.innerHeight, 0.25, 20);
			this.camera.position.set(0, 0.4, 0.7);
			scene.add( this.camera );

            // 创建相机控件
			const controls = new OrbitControls(this.camera, this.renderer.domElement);
            controls.enableDamping = true;// 阻尼器
            controls.minDistance = 0.8;// 最小距离
            controls.maxDistance = 0.8;// 最小距离
            controls.minPolarAngle = Math.PI / 2;// 相机视角垂直最小角度   目的：只能水平转动
            controls.maxPolarAngle  = Math.PI / 2;// 相机视角垂直最大角度  目的：只能水平转动
            controls.target.set(0, 0.1, 0);
            controls.update();

			scene.add( new THREE.AmbientLight( 0x404040 ) );

            // 创建点光源
			const pointLight = new THREE.PointLight( 0xffffff, 1 );
			this.camera.add( pointLight );

            // 创建渲染器通道
			const renderScene = new RenderPass( scene, this.camera );

            // Bloom发光
			const bloomPass = new UnrealBloomPass( new THREE.Vector2( window.innerWidth, window.innerHeight ), 1.5, 0.4, 0.85 );
			bloomPass.threshold = this.params.bloomThreshold;
			bloomPass.strength = this.params.bloomStrength;
			bloomPass.radius = this.params.bloomRadius;

            // 处理通道
			this.composer = new EffectComposer( this.renderer );
			this.composer.addPass( renderScene );
			this.composer.addPass( bloomPass );

            
            // 辅助线
            const axesHelper = new THREE.AxesHelper(150);
            // scene.add(axesHelper);

            const loader = new GLTFLoader().setPath('/model/');
            loader.load('people.glb', function (gltf) {
                gltf.scene.children.splice(0,1)
                that.plant = gltf.scene.children[1]
                // 创建模型的线条几何体
                let edgesGeometry = new THREE.EdgesGeometry(gltf.scene.children[0].geometry);

                // 创建线条材质
                var lineMaterial = new THREE.LineBasicMaterial({ color: 0x1d69d8, linewidth: 1 });

                // 创建线条网格
                let lineMesh = new THREE.LineSegments(edgesGeometry, lineMaterial);

                // 创建包裹对象
                let wrapper = new THREE.Object3D();
                wrapper.add(lineMesh);
                // 设置缩放比例
                let scale = 0.001; // 缩小到原始大小的一半
                wrapper.scale.set(scale, scale, scale);
                wrapper.rotateY(Math.PI);
                wrapper.position.y = 0.1;
                that.gltfChildFour = wrapper

                // 将包裹对象添加到场景中
                scene.add(wrapper);

                gltf.scene.children[1].scale.set(scale, scale, scale);
                gltf.scene.children[1].position.y = -0.06;
                // 将低盘模型添加到场景中
                scene.add(gltf.scene.children[1]);

				that.animate();

			} );

			const gui = new GUI();

			gui.add( this.params, 'exposure', 0.1, 2 ).name("发光强度").onChange( function ( value ) {

				that.renderer.toneMappingExposure = Math.pow( value, 4.0 );

			} );

			gui.add( this.params, 'bloomThreshold', 0.0, 1.0 ).name("发光阈值").onChange( function ( value ) {

				bloomPass.threshold = Number( value );

			} );

			gui.add( this.params, 'bloomStrength', 0.0, 3.0 ).name("发光强度").onChange( function ( value ) {

				bloomPass.strength = Number( value );

			} );

			gui.add( this.params, 'bloomRadius', 0.0, 1.0 ).name("发光半径").step( 0.01 ).onChange( function ( value ) {

				bloomPass.radius = Number( value );

			} );
            

			window.addEventListener( 'resize', this.onWindowResize );


        },
        onWindowResize () {
            this.camera.aspect = this.innerWidth / this.innerHeight;
            this.camera.updateProjectionMatrix();

            this.renderer.setSize(this.innerWidth, this.innerHeight);
			this.composer.setSize(this.innerWidth, this.innerHeight);
        },
        animate () {
            requestAnimationFrame(this.animate);

			this.composer.render();
            // 设置低盘转动
            if (this.plant && this.plant.position) {
                this.plant.rotateY(0.03)
            }
            this.gltfChildFour.rotateY(0.03)//整个模型转动
        },
    },
    mounted () {
        this.innerWidth = window.innerWidth
        this.innerHeight = window.innerHeight
        this.init();
    }
}
</script>
