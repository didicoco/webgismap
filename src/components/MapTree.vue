<template>
    <div class="maptree-pannel" v-show="this.$store.getters._getDefaultMapTreeVisible">
        <el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick"></el-tree>
    </div>
</template>

<script>
import { loadModules } from 'esri-loader';
const options = {
    url: 'https://js.arcgis.com/4.18/init.js',
    css: 'https://js.arcgis.com/4.18/esri/themes/light/main.css',
};

export default {
    name: 'MapTree',
    data() {
        return {
            data: [
                {
                    label: '底图数据',
                    //添加图层需要layerid、layerurl两个属性
                    children: [
                        {
                            label: '原底图',
                            layerid: 'layerid0',
                            layerurl: 'http://map.geoq.cn/arcgis/rest/services/ChinaOnlineStreetPurplishBlue/MapServer',
                        },
                        {
                            label: '暖色系图层',
                            layerid: 'layerid0',
                            layerurl: 'http://map.geoq.cn/arcgis/rest/services/ChinaOnlineStreetWarm/MapServer',
                        },
                        {
                            label: '灰色系图层',
                            layerid: 'layerid0',
                            layerurl: 'http://map.geoq.cn/arcgis/rest/services/ChinaOnlineStreetGray/MapServer',
                        },
                    ],
                },
                {
                    label: '行政区划数据',
                    children: [
                        {
                            label: '省级',
                            layerid: 'layerid1',
                            layerurl: 'https://localhost:6443/arcgis/rest/services/ChinaMap/ChinaMap_total/MapServer/2',
                        },
                        {
                            label: '市级',
                            layerid: 'layerid1',
                            layerurl: 'http://localhost:6080/arcgis/rest/services/ChinaMap/ChinaMap_total/MapServer/1',
                        },
                        {
                            label: '县级',
                            layerid: 'layerid1',
                            layerurl: 'http://localhost:6080/arcgis/rest/services/ChinaMap/ChinaMap_total/MapServer/0',
                        },
                    ],
                },
                {
                    label: '业务数据',
                    children: [
                        {
                            label: '火车站点',
                            layerid: 'layerid1',
                            layerurl: 'https://localhost:6443/arcgis/rest/services/ChinaMap/Station/MapServer',
                        },
                        {
                            label: '卷帘分析top',
                            layerid: 'swipeLayerTop',
                            layerurl: 'https://localhost:6443/arcgis/rest/services/ChinaMap/ChinaMap_total/MapServer/2',
                        },
                        {
                            label: '卷帘分析bottom',
                            layerid: 'swipeLayerBottom',
                            layerurl: 'https://localhost:6443/arcgis/rest/services/ChinaMap/ChinaMap_total/MapServer/1',
                        },
                    ],
                },
            ],
            defaultProps: {
                children: 'children',
                label: 'label',
            },
        };
    },
    methods: {
        async handleNodeClick(data) {
            switch (data.layerid) {
                case 'layerid0':
                    if (data.layerurl) {
                        const view = this.$store.getters._getDefaultView; //通过VUEX获得公共的view
                        const resultLayer = view.map.findLayerById('layerid0');
                        // console.log('resultLayer', resultLayer);
                        if (resultLayer) {
                            view.map.remove(resultLayer);
                        }
                        const [TileLayer] = await loadModules(['esri/layers/TileLayer'], options);
                        //实例化目录树中加载的图层--添加切片地图
                        const layer = new TileLayer({ url: data.layerurl, id: data.layerid, label: data.label });
                        view.map.add(layer);
                        // console.log(view.map.allLayers);
                        console.log('你加载的TileLayer图层', TileLayer);
                    }
                    break;
                case 'layerid1':
                    if (data.layerurl) {
                        const view = this.$store.getters._getDefaultView; //通过VUEX获得公共的view
                        const resultLayer = view.map.findLayerById('layerid1');
                        // console.log('resultLayer', resultLayer);
                        if (resultLayer) {
                            view.map.remove(resultLayer);
                        }
                        //实例化目录树中加载的图层--添加要素地图
                        const [FeatureLayer] = await loadModules(['esri/layers/FeatureLayer'], options);
                        const featureLayer = new FeatureLayer({
                            url: data.layerurl,
                            id: data.layerid,
                            label: data.label,
                        });
                        view.map.add(featureLayer);
                        console.log(view.map.allLayers);
                        // this.$store.commit('_setDefaultView', view);
                        console.log('你加载的FeatureLayer图层', featureLayer);
                    }
                    break;
                case 'swipeLayerTop':
                    if (data.layerurl) {
                        const view = this.$store.getters._getDefaultView; //通过VUEX获得公共的view
                        //实例化目录树中加载的图层--添加要素地图
                        const [FeatureLayer] = await loadModules(['esri/layers/FeatureLayer'], options);
                        const featureLayer = new FeatureLayer({
                            url: data.layerurl,
                            id: data.layerid,
                            label: data.label,
                        });
                        view.map.add(featureLayer);
                        console.log('你加载的FeatureLayer图层', featureLayer);
                    }
                    break;
                case 'swipeLayerBottom':
                    if (data.layerurl) {
                        const view = this.$store.getters._getDefaultView; //通过VUEX获得公共的view
                        //实例化目录树中加载的图层--添加要素地图
                        const [FeatureLayer] = await loadModules(['esri/layers/FeatureLayer'], options);
                        const featureLayer = new FeatureLayer({
                            url: data.layerurl,
                            id: data.layerid,
                            label: data.label,
                        });
                        view.map.add(featureLayer);
                        console.log('你加载的FeatureLayer图层', featureLayer);
                    }
                    break;
                default:
                    break;
            }
        },
    },
};
</script>

<style>
.maptree-pannel {
    position: absolute;
    width: 200px;
    height: 300px;
    top: 30px;
    left: 30px;
    background-color: #fff;
}
</style>
