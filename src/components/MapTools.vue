<template>
    <div class="maptools-view">
        <span class="maptools-item" @click="handleMapToolsitemClick" id="xzqh">行政区导航</span>
        <span class="maptools-item" @click="handleMapToolsitemClick" id="mls">目录树管理</span>
        <el-dropdown trigger="click" class="maptools-item" @command="handleCommand">
            <span class="el-dropdown-link"> 地图测量<i class="el-icon-arrow-down el-icon--right"></i> </span>
            <el-dropdown-menu slot="dropdown">
                <el-dropdown-item icon="el-icon-plus" command="distance">距离测量</el-dropdown-item>
                <el-dropdown-item icon="el-icon-circle-plus" command="area">面积测量</el-dropdown-item>
                <el-dropdown-item icon="el-icon-plus" command="diymeasurement_distance"
                    >自定义测量长度</el-dropdown-item
                >
                <el-dropdown-item icon="el-icon-circle-plus" command="diymeasurement_area"
                    >自定义测量面积</el-dropdown-item
                >
            </el-dropdown-menu>
        </el-dropdown>
        <el-dropdown trigger="click" class="maptools-item" @command="handleCommand">
            <span class="el-dropdown-link"> 更多功能<i class="el-icon-arrow-down el-icon--right"></i> </span>
            <el-dropdown-menu slot="dropdown">
                <el-dropdown-item icon="el-icon-search" command="spacequery">空间查询</el-dropdown-item>
                <el-dropdown-item icon="el-icon-film" command="morescreen">多屏对比</el-dropdown-item>
                <el-dropdown-item icon="el-icon-reading" command="swipanalyst">卷帘分析</el-dropdown-item>
                <el-dropdown-item icon="el-icon-printer" command="printmap">地图打印</el-dropdown-item>
                <el-dropdown-item icon="el-icon-view" command="openPopup">开启图层弹窗</el-dropdown-item>
            </el-dropdown-menu>
        </el-dropdown>
        <span class="maptools-item" @click="handleMapToolsitemClick" id="clear">清屏</span>
        <div id="dtdy"></div>
    </div>
</template>

<script>
import { loadModules } from 'esri-loader';
const options = {
    url: 'https://js.arcgis.com/4.18/init.js',
    css: 'https://js.arcgis.com/4.18/esri/themes/light/main.css',
};
export default {
    name: 'MapTools',
    methods: {
        handleMapToolsitemClick(event) {
            //  console.log(event.target.id);
            switch (event.target.id) {
                case 'xzqh': {
                    const currentVisible = this.$store.getters._getDefaultXZQHPannelVisible;
                    this.$store.commit('_setDefaultXZQHPannelVisible', !currentVisible); //行政区划的开关
                    break;
                }
                case 'mls': {
                    const currentVisible = this.$store.getters._getDefaultMapTreeVisible;
                    this.$store.commit('_setDefaultMapTreeVisible', !currentVisible); //目录树的开关
                    break;
                }
                case 'distance':
                    break;
                case 'area':
                    break;
                case 'clear':
                    this.handleClearMap();
                    break;
                default:
                    break;
            }
        },
        handleCommand(command) {
            switch (command) {
                case 'distance':
                    this.initDistanceMap();
                    break;
                case 'area':
                    this.initAreaMap();
                    break;
                case 'spacequery':
                    this.initSpaceQuery();
                    break;
                case 'morescreen':
                    this.$router.push('/onemap/one');
                    break;
                case 'swipanalyst':
                    this._initSwipe();
                    break;
                case 'printmap':
                    this.PrintMap();
                    break;
                case 'diymeasurement_distance':
                    this.initDIYMeasurement('distance');
                    break;
                case 'diymeasurement_area':
                    this.initDIYMeasurement('area');
                    break;
                case 'openPopup':
                    this.openMapPopup();
                    break;
                default:
                    break;
            }
        },
        //1、initSpaceQuery 空间查询功能
        //1.1、绘制面状区域
        async initSpaceQuery() {
            const _self = this;
            const view = _self.$store.getters._getDefaultView;
            const [SketchViewModel, Graphic, GraphicsLayer] = await loadModules(
                ['esri/widgets/Sketch/SketchViewModel', 'esri/Graphic', 'esri/layers/GraphicsLayer'],
                options,
            );

            const resultLayer = view.map.findLayerById('polygonGraphicLayer');
            if (resultLayer) {
                view.map.remove(resultLayer);
            }
            const graphicsLayer = new GraphicsLayer({
                id: 'polygonGraphicLayer',
                elevationInfo: {
                    mode: 'on-the-ground', //指定图形在垂直轴 (z) 上的放置方式,on-the-ground忽略Z值
                },
            });
            view.map.add(graphicsLayer);
            //1.2 定义面状区域的样式(符号化渲染)
            const polygonSymbol = {
                type: 'simple-fill',
                color: 'rgba(216,30,6,0.4)',
                style: 'solid',
                outline: {
                    color: '#d81e06',
                    width: 1,
                },
            };
            // 1.3 自定义草图工具
            var sketchViewModel = new SketchViewModel({
                updateOnGraphicClick: false,
                view,
                layer: graphicsLayer,
                polygonSymbol,
            });
            sketchViewModel.create('polygon'); //指定绘制的样式
            // 1.4 监听sketchViewModel的状态
            sketchViewModel.on('create-complete', function (event) {
                const graphic = new Graphic({
                    geometry: event.geometry,
                    symbol: sketchViewModel.graphic.symbol,
                });
                graphicsLayer.add(graphic);
            });
            //1.5 监听create状态，回调handleSpaceQuery方法去查询要素
            sketchViewModel.on('create', function (event) {
                if (event.state === 'complete') {
                    _self.handleSpaceQuery(event.graphic);
                }
            });
        },
        //1.6、执行空间查询的方法
        handleSpaceQuery(graphic) {
            const _self = this;
            const view = _self.$store.getters._getDefaultView;
            const resultLayer = view.map.findLayerById('layerid1');
            // const resultLayer = view.map.allLayers._items[1];(找不到ID时的替代方法)
            if (!resultLayer) {
                _self.$message({
                    message: '尚未添加业务图层，不能进行空间查询',
                    type: 'warning',
                });
                return;
            }
            const queryPoint = resultLayer.createQuery();
            queryPoint.geometry = graphic.geometry;
            resultLayer
                .queryFeatures(queryPoint)
                .then(function (results) {
                    let currentData = [];
                    if (results.features.length > 0) {
                        //符号化渲染图层
                        _self.renderResultLayer(results.features);
                        //符号化表格数据
                        results.features.map((a, b) => {
                            currentData.push({
                                name: a.attributes.name,
                                type: a.attributes.type,
                                tieluju: a.attributes.tieluju,
                                address: a.attributes.address,
                                lon: a.attributes.lon,
                                lat: a.attributes.lat,
                                key: b,
                            });
                        });
                        // console.log('currentData', currentData);
                        // console.log('results.features', results.features);
                    } else {
                        currentData.length = 0;
                    }
                    _self.$message({
                        message: `查询成功，共查到 ${results.features.length}条数据`,
                        type: 'success',
                    });
                    _self.$store.commit('_setDefaultQueryResult', currentData);
                    _self.$store.commit('_setDefaultQueryResultVisible', true);
                })
                .catch(function (error) {
                    console.log(error);
                    _self.$message.error('空间查询失败，请联系管理员');
                });
        },
        //1.7、生成查询结果图层
        async renderResultLayer(resultFeatures) {
            const view = this.$store.getters._getDefaultView;
            const [FeatureLayer] = await loadModules(['esri/layers/FeatureLayer'], options);
            const resultLayer = view.map.findLayerById('initResultLayer');
            if (resultLayer) view.map.remove(resultLayer);
            const resultData = this._translateLonLat(resultFeatures);
            //实例化弹窗
            let template = {
                title: '{name}-{tieluju}',
                content: [
                    {
                        type: 'fields',
                        fieldInfos: [
                            {
                                fieldName: 'name',
                                label: '名称',
                            },
                            {
                                fieldName: 'type',
                                label: '类型',
                            },
                            {
                                fieldName: 'tieluju',
                                label: '铁路局',
                            },
                            {
                                fieldName: 'address',
                                label: '地址',
                            },
                        ],
                    },
                ],
            };
            const queryResultLayer = new FeatureLayer({
                source: resultData,
                id: 'initResultLayer',
                ObjectIdField: 'ObjectID',
                renderer: {
                    type: 'simple',
                    symbol: {
                        type: 'picture-marker',
                        url: `static/icon/train.png`,
                        width: '32px',
                        height: '32px',
                    },
                },
                fields: [
                    {
                        name: 'OBJECTID',
                        type: 'oid',
                    },
                    {
                        name: 'name',
                        type: 'string',
                    },
                    {
                        name: 'type',
                        type: 'string',
                    },
                    {
                        name: 'tieluju',
                        type: 'string',
                    },
                    {
                        name: 'address',
                        type: 'string',
                    },
                ],
                popupTemplate: template,
            });
            view.map.add(queryResultLayer);
        },
        //1.8、查询数据遍历
        _translateLonLat(data) {
            const _self = this;
            if (data.length > 0) {
                _self.geoData = [];
                data.map((value, key) => {
                    _self.geoData.push({
                        geometry: {
                            //geometry的结构是固定死的,必须要有这3项
                            type: 'point',
                            x: Number(value.attributes.lon),
                            y: Number(value.attributes.lat),
                        },
                        attributes: {
                            ObjectID: key + 1,
                            name: value.attributes.name,
                            type: value.attributes.type,
                            tieluju: value.attributes.tieluju,
                            address: value.attributes.address,
                        },
                    });
                });
                // console.log(_self.geoData);
            }
            return _self.geoData;
        },
        //2、卷帘分析功能
        async _initSwipe() {
            const _self = this;
            const view = _self.$store.getters._getDefaultView;
            const [Swipe] = await loadModules(['esri/widgets/Swipe'], options);
            const topLayer = view.map.findLayerById('swipeLayerTop');
            const bottomLayer = view.map.findLayerById('swipeLayerBottom');
            if (topLayer && bottomLayer) {
                _self.swipe = new Swipe({
                    leadingLayers: [topLayer],
                    trailingLayers: [bottomLayer],
                    position: 35,
                    view: view,
                    direction: 'vertical',
                });
                view.ui.add(_self.swipe);
            } else {
                _self.$message({
                    message: '请添加至少两张业务图层',
                    type: 'warning',
                });
                return;
            }
        },
        //3、地图打印功能
        async PrintMap() {
            const _self = this;
            const view = _self.$store.getters._getDefaultView;
            const [PrintTask, PrintTemplate, PrintParameters] = await loadModules(
                ['esri/tasks/PrintTask', 'esri/tasks/support/PrintTemplate', 'esri/tasks/support/PrintParameters'],
                options,
            );
            let printTask = new PrintTask({
                url: 'https://utility.arcgisonline.com/arcgis/rest/services/Utilities/PrintingTools/GPServer/Export%20Web%20Map%20Task',
            });
            let template = new PrintTemplate({
                format: 'pdf',
                exportOptions: {
                    dpi: 100,
                },
                layout: 'a3-portrait',
                layoutOptions: {
                    titleText: '地图出图demo',
                    authorText: 'hd',
                    scalebarUnit: 'Kilometers',
                },
            });
            let params = new PrintParameters({
                view: view,
                template: template,
            });
            printTask.execute(params).then((printResult, printError) => {
                console.log(printResult, printError);
                if (printResult.url) window.open(printResult.url);
                if (printError) this.$message.error('地图打印失败');
            });
        },

        //4、地图测量功能
        async initDistanceMap() {
            const _self = this;
            const view = _self.$store.getters._getDefaultView;
            const [DistanceMeasurement2D] = await loadModules(['esri/widgets/DistanceMeasurement2D'], options);
            if (this.measurementWidge) this.measurementWidge.destroy();
            this.measurementWidge = new DistanceMeasurement2D({
                view,
            });
            view.ui.add(this.measurementWidge, 'top-left');
        },
        //5、面积测量功能
        async initAreaMap() {
            const _self = this;
            const view = _self.$store.getters._getDefaultView;
            const [AreaMeasurement2D] = await loadModules(['esri/widgets/AreaMeasurement2D'], options);
            if (this.measurementWidge) this.measurementWidge.destroy();
            this.measurementWidge = new AreaMeasurement2D({
                view,
            });
            view.ui.add(this.measurementWidge, 'top-left');
        },
        //6、自定义测量功能
        async initDIYMeasurement(type) {
            const _self = this;
            const view = _self.$store.getters._getDefaultView;
            const resultLayer = view.map.findLayerById('measurementGraphicLayer');
            if (resultLayer) view.map.remove(resultLayer);
            const [
                SketchViewModel,
                Graphic,
                GraphicsLayer,
                GeometryService,
                LengthsParameters,
                AreasAndLengthsParameters,
            ] = await loadModules(
                [
                    'esri/widgets/Sketch/SketchViewModel',
                    'esri/Graphic',
                    'esri/layers/GraphicsLayer',
                    'esri/tasks/GeometryService',
                    'esri/tasks/support/LengthsParameters',
                    'esri/tasks/support/AreasAndLengthsParameters',
                ],
                options,
            );
            const graphicsLayer = new GraphicsLayer({
                id: 'measurementGraphicLayer',
                elevationInfo: {
                    mode: 'on-the-ground',
                },
            });
            view.map.add(graphicsLayer);

            if (type === 'distance') {
                const polylineSymbol = {
                    type: 'simple-line',
                    color: '#d81e06',
                    width: '2',
                    style: 'solid',
                };
                const sketchViewModel = new SketchViewModel({
                    updateOnGraphicClick: false,
                    view: view,
                    layer: graphicsLayer,
                    polylineSymbol,
                });
                sketchViewModel.create('polyline');
                sketchViewModel.on('create-complete', function (event) {
                    const graphic = new Graphic({
                        geometry: event.geometry,
                        symbol: sketchViewModel.graphic.symbol,
                    });
                    graphicsLayer.add(graphic);
                });
                sketchViewModel.on('create', function (event) {
                    if (event.state === 'complete') {
                        console.log(graphicsLayer);
                        console.log(event);

                        //获取线段长度
                        const geometryService = new GeometryService({
                            url: 'http://sampleserver6.arcgisonline.com/arcgis/rest/services/Utilities/Geometry/GeometryServer',
                        });
                        const lengthsParameters = new LengthsParameters();
                        lengthsParameters.polylines = event.graphic.geometry;
                        lengthsParameters.lengthUnit = 9036; //9036代表公里(km)的编码,9001:m
                        lengthsParameters.geodesic = true;

                        geometryService.lengths(lengthsParameters).then(function (result) {
                            console.log('长度', result);
                        });
                    }
                });
            } else if (type === 'area') {
                const polygonSymbol = {
                    type: 'simple-fill',
                    color: 'rgba(216,30,6,0.4)',
                    style: 'solid',
                    outline: {
                        color: '#d81e06',
                        width: 1,
                    },
                };
                const sketchViewModel = new SketchViewModel({
                    updateOnGraphicClick: false,
                    view: view,
                    layer: graphicsLayer,
                    polygonSymbol,
                });
                sketchViewModel.create('polygon');

                sketchViewModel.on('create-complete', function (event) {
                    const graphic = new Graphic({
                        geometry: event.geometry,
                        symbol: sketchViewModel.graphic.symbol,
                    });
                    graphicsLayer.add(graphic);
                });
                sketchViewModel.on('create', function (event) {
                    if (event.state === 'complete') {
                        console.log('graphicsLayer', graphicsLayer);
                        console.log(event);
                        //获取周长和面积
                        const geometryService = new GeometryService({
                            url: 'https://sampleserver6.arcgisonline.com/arcgis/rest/services/Utilities/Geometry/GeometryServer',
                        });
                        const areasAndLengthsParameters = new AreasAndLengthsParameters();
                        areasAndLengthsParameters.polygons = event.graphic.geometry;
                        areasAndLengthsParameters.areaUnit = 'square-kilometers';
                        areasAndLengthsParameters.lengthUnit = 'kilometers';

                        geometryService.areasAndLengths(areasAndLengthsParameters).then(function (result) {
                            console.log('周长和面积', result);
                        });
                    }
                });
            }
        },
        //7、开启图层弹窗功能
        openMapPopup() {
            const _self = this;
            const view = _self.$store.getters._getDefaultView;
            const resultLayer = view.map.findLayerById('layerid1');
            if (resultLayer) {
                view.on('click', function (event) {
                    view.hitTest(event).then(function (response) {
                        console.log('1', response);
                        if (response.results.length) {
                            let graphic = response.results.filter(function (result) {
                                return result.graphic.layer.id === 'layerid1';
                            })[0].graphic;
                            console.log(graphic.attributes);
                        }
                    });
                });
            } else {
                _self.$message({
                    message: '请添加业务图层',
                    type: 'warning',
                });
            }
        },
        //8、清屏功能
        handleClearMap() {
            const view = this.$store.getters._getDefaultView;
            const resultLayer1 = view.map.findLayerById('swipeLayerTop');
            const resultLayer2 = view.map.findLayerById('swipeLayerBottom');
            const resultLayer3 = view.map.findLayerById('layerid1');
            const resultLayer4 = view.map.findLayerById('measurementGraphicLayer');
            if (resultLayer1) view.map.remove(resultLayer1);
            if (resultLayer2) view.map.remove(resultLayer2);
            if (resultLayer3) view.map.remove(resultLayer3);
            if (resultLayer4) view.map.remove(resultLayer4);
            if (this.swipe) this.swipe.destroy();
            if (this.measurementWidget) this.measurementWidget.destroy();
        },
    },
};
</script>

<style>
.maptools-view {
    position: absolute;
    height: 30px;
    padding: 0 15px;
    top: 20px;
    right: 15px;
    background-color: #fff;
}
.maptools-item {
    line-height: 30px;
    margin-left: 15px;
    color: black;
    cursor: pointer;
    /* cursor:pointer属性是在计算机中将光标呈现为指示链接的指针（一只手）  */
}
.maptools-item:first-child {
    margin-left: 0;
}
.el-dropdown-link {
    font-size: 16px;
}
#dtdy {
    position: absolute;
    top: 200px;
    left: 200px;
}
</style>
