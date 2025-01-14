<template>
    <div id="my-three"></div>
</template>
<script setup>
import { ref, onMounted } from 'vue';
import * as THREE from 'three'
import { OrbitControls } from 'three/examples/jsm/controls/OrbitControls'
const pageBg = ref(require('../assets/three-bg.jpeg'))
console.log(pageBg.value)
const onInit = () => {
    //创建一个三维场景
    const scene = new THREE.Scene();
    // const texture = new THREE.TextureLoader().load(pageBg.value);
    // texture.mapping = THREE.EquirectangularReflectionMapping
    // scene.background = texture
    // scene.environment = texture

    let picList = ["right","left","top","bottom","front","back"];
    var material2 = []
    
    for(let i = 0;i < picList.length;i++){
        const url = require(`../assets/bg-${picList[i]}.jpg`)
        console.log(url)
        material2.push(url)
    }
    scene.background= new THREE.CubeTextureLoader().load(material2);

    //创建一个物体（形状）
    const geometry = new THREE.BoxGeometry(200, 100, 80);//长宽高都是100的立方体
    // // const geometry = new THREE.SphereGeometry(60,40,40);//半径是60的圆
    // //widthSegments, heightSegments：水平方向和垂直方向上分段数。widthSegments最小值为3，默认值为8。heightSegments最小值为2，默认值为6。
    //创建材质（外观）
    function createMaterial(color1,color2){
        
        var attributes = {};
        var uniforms = {
            time:{type:'f',value:0.2},
            scale:{type:'f',value:0.2},
            alpha:{type:'f',value:0.6},
            resolution:{type:"v2",value:new THREE.Vector2()},
            color1: {
                value: new THREE.Color(color1)
            },
            color2: {
                value: new THREE.Color(color2)
            }  
        };
    
        var meshMaterial = new THREE.ShaderMaterial({
            uniforms:uniforms,
            attributes:attributes,
            vertexShader:`
                varying vec2 vUv;

                void main() {
                vUv = uv;
                gl_Position = projectionMatrix * modelViewMatrix * vec4(position,1.0);
                }
            `,
            fragmentShader:`
                uniform vec3 color1;
                uniform vec3 color2;
            
                varying vec2 vUv;
                
                void main() {
                
                gl_FragColor = vec4(mix(color1, color2, vUv.y), 1.0);
                }
            `,
            transparent:true        
            });
        return meshMaterial;
    }
    const material = [
        createMaterial('#5ee7df', '#fedge3'), // 前面 
        createMaterial('#409eff', '#ecf5ff'), // 后面 
        createMaterial('#67c23a', '#f0f9eb'), // 上面 
        createMaterial('#e6a23c', '#fdf6ec'), // 下面 
        createMaterial('#f56c6c', '#fef0f0'), // 左面 
        createMaterial('#b515d1', '#f8e2fc')  // 右面 
    ]
    //创建一个网格模型对象
    const mesh = new THREE.Mesh(geometry, material);//网络模型对象Mesh
    //把网格模型添加到三维场景
    scene.add(mesh);//网络模型添加到场景中

    // 添加多个模型（添加圆形）
    // const  geometry2 = new THREE.SphereGeometry(60, 40, 40);
    // const  material2 = new THREE.MeshLambertMaterial({
    //     color: 0xffff00
    // });
    // const mesh2 = new THREE.Mesh(geometry2, material2); //网格模型对象Mesh
    // // mesh3.translateX(120); //球体网格模型沿Y轴正方向平移120
    // mesh2.position.set(120,0,0);//设置mesh3模型对象的xyz坐标为120,0,0
    // scene.add(mesh2);

    //添加光源 //会照亮场景里的全部物体（氛围灯），前提是物体是可以接受灯光的，这种灯是无方向的，即不会有阴影。
    const ambient = new THREE.AmbientLight(0xffffff, 0.4);
    const light = new THREE.PointLight(0xffffff, 1);//点光源，color:灯光颜色，intensity:光照强度

    scene.add(ambient);
    light.position.set(200,300,400);
    scene.add(light);
    
    //创建一个透视相机，窗口宽度，窗口高度
    const width = window.innerWidth, height = window.innerHeight;
    const camera = new THREE.PerspectiveCamera(45, width/height, 1, 1000);
    //设置相机位置
    camera.position.set(300,300,300);
    //设置相机方向
    camera.lookAt(0,0,0);

    // //创建辅助坐标轴
    // const axesHelper = new THREE.AxesHelper(200);//参数200标示坐标系大小，可以根据场景大小去设置
    // scene.add(axesHelper);

    //创建一个WebGL渲染器
    const renderer = new THREE.WebGLRenderer({antialias: true, alpha: true})

    renderer.setSize(width,height)//设置渲染区尺寸
    renderer.render(scene,camera)//执行渲染操作、指定场景、相机作为参数

    const controls = new OrbitControls(camera, renderer.domElement)//创建控件对象
    controls.addEventListener('change',()=>{
        renderer.render(scene, camera)//监听鼠标，键盘事件
    })
    document.getElementById('my-three')?.appendChild(renderer.domElement)
}
onMounted(() => {
    onInit()
})
</script>
<style scoped>
    #my-three {
        width: 100vw;
        height: 100vh;
        /* background-image: url('../assets/three-bg.jpeg');
        background-size: 100% 100%; */
    }
</style>