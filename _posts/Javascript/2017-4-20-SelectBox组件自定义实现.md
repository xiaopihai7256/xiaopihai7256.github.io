---
layout: post
title: SelectBox组件自定义实现
categories: 
 - JavaScript
author: 杨比轩
---

## 1.实现效果展示：

![UI](http://upload-images.jianshu.io/upload_images/1156415-fe10074536b7e280.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

功能：Oracle From builder一个很常见的组件，类似的功能计时左侧一列为待选择项，右侧一列的已选择项。选中的选中会从左侧移到右侧，取消的选项会从右侧返回到左侧。事件包括双击选择和取消，支持多选操作，并通过Kendo UI的DataSource来进行数据的交互，包括组件的初始化和数据的编辑保存等等。

## 2.实现思路：
### 2.1 UI实现
UI的实现完全基于Html5，布局使用了bootstrap，除此之外没有依赖其他组件。
```html
<div class="col-md-8">
    <div class="col-md-4">
        <select id="un-select" ondblclick="moveTo()" multiple=”multiple” size="6" class="select-bar">
                                                </select>
    </div>

    <div class="col-md-2">
        <div class="row" style="text-align: center">
            <button id="move-right" class="btn btn-primary" onclick="moveTo()" style="margin-bottom: 20px;width: auto">
                                                        <span class="glyphicon glyphicon-chevron-right"></span>
                                                    </button>
        </div>
        <div class="row" style="text-align: center">
            <button id="move-left" class="btn btn-warning" onclick="moveBack()" style="width: auto">
                                                        <span class="glyphicon glyphicon-chevron-left"></span>
                                                    </button>
        </div>

    </div>

    <div class="col-md-4">
        <select id="already-select" ondblclick="moveBack()" multiple=”multiple” class="select-bar" size="6">
                                                </select>
    </div>

    <div class="col-md-1">
        <div class="row">
            <button id="lock-position" class="btn btn-primary" style="margin: 10px" onclick="savePosition()" name="to">
                                                    <@spring.message "hap.confirm"/>
                                                    </button>
        </div>
        <div class="row">
            <button class="btn btn-success" style="margin: 10px" onclick="unlockPosition()" name="to">
                                                    <@spring.message "hpm.setup.type.unlock"/>
                                                    </button>
        </div>
    </div>
</div>
```

### 2.2 逻辑实现
整个逻辑的实现分为两部分，一部分的数据的管理，一部分为事件的响应。

#### 数据的管理
数据的管理由于是基于kendo UI的DataSource，所以支持MVVM，在初始的时候，从服务器拉取数据对组件进行初始化，并依靠DataSource內建的方法进行管理，包括数据的拉取，同步，添加删除，移动更改等等。

代码除了项目中规定的一些操作逻辑外，核心的工作就是实现DataSource的数据和UI数据的绑定更新响应工作，使之完全使用kendo UI的內建方法实现数据的同步，减少工作量。同时也不额外的引进其他的框架和组件。

具体实现的代码中有详细的注解，可以参考代码进行理解。

```javascript
//选中方法实现
 function moveTo(){

        if(viewModel.model.typeId == null){
            kendo.ui.showInfoDialog({
                message: '<@spring.message "hpm.setup.type.chooseone"/>'
            });
            return;
        }

        //添加数据分两种：1.第一次添加（数据源中没有相应记录） ，2.撤销之前的操作（数据源中有记录，标志位为DELETE）

        //遍历数据源，没有则第一次添加，有则修改标志位
        var datas = positionDataSource.data();

        var selectData = $('#un-select option:selected');
        for(var j = 0;j<selectData.length;j++){
            var oValue = selectData.eq(j).val();
        for(var i = 0,len=datas.length; i < len; i++) {
            data = datas[i];
            if(data.positionCode === oValue){
                data.set("tag",null);
                //ui响应
                $('#already-select').prepend($('#un-select option:selected'));
                return;
            }
        }
        //遍历之后没有找到相应的数据，则进行新增
        positionDataSource.add({typeId:viewModel.model.typeId,tag:"NEW",positionCode:oValue});
        }
        //ui响应
        $('#already-select').prepend($('#un-select option:selected'));

    }

//移除方法实现
    function moveBack() {

        if(viewModel.model.typeId == null){
            kendo.ui.showInfoDialog({
                message: '<@spring.message "hpm.setup.type.chooseone"/>'
            });
            return;
        }

        //移除数据分两种，1.原来的进行移除(标志位为null或者未定义)，2.新增数据进行移除(标志位为NEW)
        var datas = positionDataSource.data();

        var selectData = $('#already-select option:selected');
        for(var j = 0;j<selectData.length;j++) {
            var oValue = selectData.eq(j).val();
            for (var i = 0, len = datas.length; i < len; i++) {
                data = datas[i];
                if (data.positionCode === oValue) {
                    if (data.tag == null) {
                        data.set("tag", "DELETE");
                    }
                    if (data.tag === "NEW") {
                        //数据还没有同步到后台，所以只需要在前端删除即可
                        positionDataSource.remove(data);
                    }
                }
            }

        }
        $('#un-select').prepend($('#already-select option:selected'));
    }

//全部移除方法，二次初始化中默认会进行调用
    function moveBackAll() {

        $('#un-select').prepend($("#already-select option"));
    }

//支持多选操作
    function selectDataItem() {
        //获取选择到的数据
        var treeList = $("#treeList").data("kendoTreeList");
        var row = treeList.select();
        return treeList.dataItem(row);
    }

//数据更新到DataSource中
    function updateDataToSource(dataItem){
        if(dataItem === undefined || dataItem === null){
            return null;
        }
        var formData = viewModel.model.toJSON();
        for ( var k in formData) {
            dataItem.set(k, viewModel.model[k]);
        }
    }

//初始化组件
    function initializeSelect(projectPosition){

        var unSelect = $("#un-select");
        for(var i = 0,len=projectPosition.length; i < len; i++) {
            data = projectPosition[i];
            unSelect.append("<option value='" + data.value + "'>" + data.meaning + "</option>");
        }

    }

//载入组件
    function loadSelect() {

        //由于此处要对datas进行迭代，所以必须read()到数据后才进行，因此positionDataSource的read关闭异步
        positionDataSource.read();
        var datas = positionDataSource.data();
        var alreadySelect = $('#already-select');

        for(var i = 0,len=datas.length; i < len; i++) {
            data = datas[i];
            alreadySelect.prepend($("#un-select option[value=" + data.positionCode + "]"));
        }
    }

//保存逻辑
    function savePosition() {
        if(viewModel.model.typeId == null || viewModel.model.typeId === ""){
            kendo.ui.showInfoDialog({
                message: '<@spring.message "hpm.setup.type.chooseone"/>'
            });
            return;
        }

        //按照标志位对数据进行操作,新增的不做处理，delete的直接删除再同步
        var datas = positionDataSource.data();

        var deleteData = datas.filter(function(item){
            return item.tag === 'DELETE';
        });

        $.each(deleteData,function(i,v){
            positionDataSource.remove(v)
        });

        positionDataSource.sync();

        lockPosition();
    }

//锁定逻辑
    function lockPosition() {
        //设置锁定标志位为Y
        viewModel.model.set("teamMemberFlag","Y");
        setOld();

        //添加禁止属性，并移除按钮绑定的事件
        $('#un-select').attr("disabled","disabled");
        $('#already-select').attr("disabled","disabled");
        $('#move-right').removeAttr("onclick");
        $('#move-left').removeAttr("onclick");
        $('#lock-position').removeAttr("onclick");

        $('#move-right').attr("disabled","disabled");
        $('#move-left').attr("disabled","disabled");
        $('#lock-position').attr("disabled","disabled");

        //修改相应样式
        $("#un-select").css("background-color","#e4e4e4");
        $("#already-select").css("background-color","#e4e4e4");

    }

//解锁逻辑
    function unlockPosition() {

        if(viewModel.model.typeId == null){
            kendo.ui.showInfoDialog({
                message: '<@spring.message "hpm.setup.type.chooseone"/>'
            });
            return;
        }

        //设置锁定标志位为Y
        viewModel.model.set("teamMemberFlag","N");
        setOld();

        $('#move-right').attr("onclick","moveTo()");
        $('#move-left').attr("onclick","moveBack()");
        $('#lock-position').attr("onclick","savePosition()");
        $('#un-select').removeAttr("disabled");
        $('#already-select').removeAttr("disabled");

        $('#move-right').removeAttr("disabled");
        $('#move-left').removeAttr("disabled");
        $('#lock-position').removeAttr("disabled");

        //修改相应样式
        $("#un-select").css("background-color","#ffffff");
        $("#already-select").css("background-color","#ffffff");
    }

```