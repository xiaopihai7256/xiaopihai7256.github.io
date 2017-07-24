---
layout: post
title: freeMark动态生成属性组
categories: 
 - FreeMark
author: 杨比轩
---

项目开发中遇到的一个实际且实用的功能，就是对应一个项目数据，需要配套的属性组数据，这一系列的属性组数据是对项目数据的补充，说白了就是一个可以拓展该项目的其它信息，来进行保存。

## 实际需求
一个项目，可能对对应多个属性组，属性组的展示方式和一些特性可以设置，根据设置的特性来动态的生成该模块，需要实现的功能是：
  1. 属性组可能是表单或者表格，但是无论哪种展示方法，实际的数据都是保存在同一张表中
  2. 属性可能为空，也可能为一个或者多个
  3. 必须以Tab页的方式展示
  4.每一个属性组都可以单独的进行编辑，更新，保存，删除等操作

## 问题分析
技术的大方向是固定的，即使用freeMark模板技术来进行动态生成前端代码，包括UI和js的逻辑代码。

特别注意点：
 整个功能需要在系统中的多个页面进行调用，所以需要抽象为一个独立的顶层模块，并且不是实际依赖实际页面的数据。所以最后抽象为一个java方法，将必要的数据以参数的形式传递，然后加载ftl模板，生成前端代码，最后添加到请求的页面中去，完成整个逻辑。

```javascript
//前端调用形式
${attrGroupProvider.getAttrGroup(base,"hpm.impl.dto.Project","projectId","${projectId!0}",attributeGroups)}
```
实际开发细节问题的思路：
```
思路整理分析
---

1. 根据提供的接口，拿到该项目设置的属性组头表ID结合
2. 拿到集合，查到集合对应的属性组的集合，存储进

    问题1：多个集合组，数据存储形式

    > 　解决思路：map<属性头表dto,List<属性dto>>

3. java加载ftl模板，根据参数，初始化出来界面‘

    - map不为空，就建立tab标签页组件
    - 遍历map，创建tab页头和对应的div
    - 根据属性组头类型属性，确定加载grid或者form组
    - form组值遍历循环，添加为input即可

   问题2：拿到属性组加载对应的grid
    
    > 属性头id-》属性组配置-》配置初始化grid和数据源的代码

4. 页面初始化完成，进行数据拉取绑定到viewModel，流程完成



项目新建页面整体流程
---
请求方法controller请求，三种情况

- 项目id
    
    项目id是查看项目明细数据，先获取模板id，初始化viewAndModel，获取项目明细数据配置字段的数据，初始化项目明细数据的配置字段显示

    接下来是属性组配置，此时有项目id，获取到项目的dto，调用属性组接口，获取数据，初始化配置数据页面


- 模板id

    模板id，获取模板配置数据，初始化页面，拉取默认字段

    获取属性组，同上

- lineId

    行表id，获取lineDto，获取模板id，加载模板数据
```

## 模板代码：
java部分
```java
    public String getAttrGroupByDto(RequestContext requestContext,String dtoFullName,String pkName,String primaryKey) throws IOException, TemplateException{

        //根据条件获取要展示的属性组
        List<AttributesGroup> attributeGroups = null;
        try {
            String str = dtoFullName.replace("dto", "service");
            String serviceName = str.substring(0, str.lastIndexOf(".") + 1) + "I" + str.substring(str.lastIndexOf(".") + 1) + "Service";
            String serviceImplName = serviceName.substring(serviceName.lastIndexOf(".")+2).substring(0,1).toLowerCase() + serviceName.substring(serviceName.lastIndexOf(".")+2).substring(1) + "Impl";
            if(attributeGroups ==  null ){
                Class<?> dtoClass = Class.forName(dtoFullName);
                Object dtoObj = dtoClass.newInstance();
                WebApplicationContext wac = ContextLoader.getCurrentWebApplicationContext();

                Object serviceImpl = wac.getBean(serviceImplName, Class.forName(serviceName));
                Method selectByPrimaryKey = serviceImpl.getClass().getDeclaredMethod(QUERY_METHOD_NAME, dtoClass);
                if(!"".equals(primaryKey)) {
                    Field nameField = dtoClass.getDeclaredField(pkName);
                    nameField.setAccessible(true);
                    nameField.set(dtoObj, Long.parseLong(primaryKey));
                }
                Object resultDto = selectByPrimaryKey.invoke(serviceImpl,dtoObj);
                if(resultDto == null){
                    attributeGroups = attributesAssignService.selectGroupsByBaseDTO((BaseDTO) dtoObj);
                }else
                {
                    attributeGroups = attributesAssignService.selectGroupsByBaseDTO((BaseDTO) resultDto);
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
            return "<p>属性组查询失败</p>";
        }

        if(attributeGroups == null){
            return "";
        }

        return getAttrGroup(requestContext,dtoFullName,pkName,primaryKey,attributeGroups);

    }

    public String getAttrGroup(RequestContext requestContext,String dtoFullName,String pkName,String primaryKey,List<AttributesGroup> attributeGroups) throws IOException, TemplateException{

        String extUrl = dtoFullName.substring(dtoFullName.lastIndexOf(".")+1).substring(0,1).toLowerCase()+ dtoFullName.substring(dtoFullName.lastIndexOf(".")+1).substring(1)+"Ext";

        if(attributeGroups == null){
            return "";
        }

        Template template = getConfiguration().getTemplate("AttributeGroup.ftl");

        try(StringWriter out = new StringWriter()){

            Map<String, Object> param = new HashMap<>();

            Map<String, String> lovDatas = new HashMap<>();

            Map<String, List<AttributesCols>> groupsCols = new HashMap<>();

            List<String> codeResources = new ArrayList();

            AttributesCols attributesCols = new AttributesCols();

            for(AttributesGroup group : attributeGroups){

                attributesCols.setAttributeGroupId(group.getAttributeGroupId());
                List<AttributesCols> attributeRestoutCols=  attributesColsService.selectColsByGroupId(attributesCols);
                for(AttributesCols col:attributeRestoutCols){
                    if("LOV".equals(col.getFormatCode())){
                        if(col.getFormatValue() != null){
                            String lovData = kendoLovService.getLov(requestContext.getContextPath(),requestContext.getLocale(),col.getFormatValue());
                            lovDatas.put(col.getFormatValue(),lovData);
                        }
                    };
                    if("LIST".equals(col.getFormatCode())){
                        if(col.getFormatValue() != null) {
                            codeResources.add(col.getFormatValue());
                        }
                    }
                }
                groupsCols.put(group.getAttributeGroupId().toString(),attributeRestoutCols);
            }

            param.put("groupsCols", groupsCols);
            param.put("attributeGroups", attributeGroups);
            param.put("lovDatas", lovDatas);
            param.put("base", requestContext);
            param.put("listSources", codeResources);
            param.put("pkName", pkName);
            param.put("dtoFullName", dtoFullName);
            param.put("primaryKey", primaryKey);
            param.put("extUrl", extUrl);
            template.process(param, out);
            out.flush();
            String html = out.toString();
            return html;

        }

    }
```
freeMark部分
```freeMark
<#-- 属性组动态配置模板 -->

<div class="row" >
    <div class="col-md-12">
        <h4>属性组</h4>
        <hr/>
    </div>
</div>
<#-- 下拉列表取值源 -->
<#if listSources?? >
    <#list listSources as ls>
        <script src="${base.contextPath}/common/code?${ls+ "Resource"}=${ls}"type="text/javascript"></script>
    </#list>
</#if>
<#if groupsCols?? >
<#-- 属性组数据存在，遍历创建tab页 -->
<script>

    var groupModel = kendo.observable({
    <#-- 创建form绑定对象 -->
        <#list attributeGroups as group>

            <#if group.displayType == "FORM">

            ${group.groupInternalName}${group.attributeGroupId}:{
                ${pkName}:${primaryKey!null},
                attributeGroupId:${group.attributeGroupId},
                ext1:2
            },

            <#elseif group.displayType == "GRID">
                <#-- 生成grid的操作方法，包括新增，保存，取消-->
                ${"grid" + group_index + "Insert"}:function(e){

                    if(typeof(viewModel.model.${pkName}) === "undefined"){
                        kendo.ui.showInfoDialog({
                            message: '请先保存，再进行属性组操作!'
                        });

                        return;
                    }

                    ${"grid" + group_index}.addRow();
                },
                ${"grid" + group_index + "Save"}:function(e){

                    if(typeof(viewModel.model.${pkName}) === "undefined"){
                        kendo.ui.showInfoDialog({
                            message: '请保存先保存项目，再进行属性组操作!'
                        });

                        return;
                    }

                    ${"grid" + group_index}.saveChanges();
                },
                ${"grid" + group_index + "Cancel"}:function(e){

                    if(typeof(viewModel.model.${pkName}) === "undefined"){
                        kendo.ui.showInfoDialog({
                            message: '请保存先保存项目，再进行属性组操作!'
                        });

                        return;
                    }

                    ${"grid" + group_index}.cancelChanges();
                },
            </#if>
        </#list>

        //以下所有操作，必须保证当前页面的viewModel里有pkName，否则视为当前没有已创建项目，会提示：请保存先保存项目，再进行属性组操作
        groupFromsRead:function () {
            if(typeof(viewModel.model.${pkName}) === "undefined" || viewModel.model.${pkName} == null){
                return;
            }
            //构造 查询条件参数
            var groups = [
            <#list attributeGroups as group>
                <#if group.displayType == "FORM">
                    <#if (attributeGroups?size)-1 == group_index >
                        {
                            ${pkName}:viewModel.model.${pkName},
                            attributeGroupId:${group.attributeGroupId}
                        }
                    <#else>
                        {
                            ${pkName}:viewModel.model.${pkName},
                            attributeGroupId:${group.attributeGroupId}
                        },
                    </#if>
                </#if>
            </#list>
            ];

            $.ajax({
                url:"${base.contextPath}/attr/${extUrl}/formQuery",
                contentType: "application/json",
                type: "POST",
                data:kendo.stringify(groups),
                success:function(data){

                    //将数据绑定到form中去
                    var formatModel = groupModel.toJSON();
                    var formatGroup;
                    var group;
                    for(var modelKey in formatModel){
                        group = groupModel[modelKey];
                        var remoteData = data[group.attributeGroupId.toString()];
                        formatGroup = group.toJSON();
                        for(var groupKey in remoteData){
                            if(remoteData[groupKey] != null){
                                group.set(groupKey,remoteData[groupKey]);
                            }
                        }
                    }
                }
            })
        },
        //新建和更新
        GroupFromSync:function (formKey) {

            if(typeof(viewModel.model.${pkName}) === "undefined"){
                kendo.ui.showInfoDialog({
                    message: '请保存先保存项目，再进行属性组操作!'
                });

                return;
            }

            if(viewModel.model.${pkName} == null){
                kendo.ui.showInfoDialog({
                    message: '请保存先保存项目，再进行属性组操作!'
                });

                return;
            }

            var groupFormValidateResult = $('#'+formKey).kendoValidator({
                valid: function (e) {
                },
                //验证样式 默认为default
                invalidMessageType : "tooltip"
            }).data("kendoValidator").validate();


           if(!groupFormValidateResult){
                // num数字输入组件样式加载会出问题，在这里进行修复
                var Numerics = $('#'+formKey).find('.k-numeric-wrap');
               for(var i = 0,len = Numerics.length; i < len; i++){
                   Numeric = Numerics[i];
                   var child  = $($(Numeric).children()[1]);
                   if((child.val() === undefined || child.val() === null || child.val() === "") && child.attr("required") === "required"){
                       $(Numeric).addClass("k-valid-custom");
                   }else{
                       $(Numeric).removeClass("k-valid-custom");
                   }
               }

               // LOV输入组件样式加载会出问题，在这里进行修复
               var Lovs = $('#'+formKey).find('.k-lov');
               for(var i = 0,len = Lovs.length; i < len; i++){
                   Lov = Lovs[i];
                   childSpan  = $(Lov).children()[0];
                   var childInput  = $($(Lov).children()[1]);
                   if((childInput.val() === undefined || childInput.val() === null || childInput.val() === "") && childInput.attr("required") === "required"){
                       $(childSpan).addClass("k-valid-custom");
                   }else{
                       $(childSpan).removeClass("k-valid-custom");
                   }
               };

               // LOV输入组件样式加载会出问题，在这里进行修复
               var Lists = $('#'+formKey).find('.k-combobox');
               for(var i = 0,len = Lists.length; i < len; i++){
                   list = Lists[i];
                   childSpan  = $(list).children()[0];
                   var childInput  = $($(list).children()[1]);
                   if((childInput.val() === undefined || childInput.val() === null || childInput.val() === "") && childInput.attr("required") === "required"){
                       $(childSpan).addClass("k-valid-custom");
                   }else{
                       $(childSpan).removeClass("k-valid-custom");
                   }
               }
                return;
            }

            var groupData = this[formKey];
            groupData.set("${pkName}",viewModel.model.${pkName});

            $.ajax({
                url:"${base.contextPath}/attr/${extUrl}/formSubmit",
                contentType: "application/json",
                type: "POST",
                dataType:"JSON",
                data:kendo.stringify(groupData),
                success:function(data){

                    if(data.insert === true || data.update === true){
                        kendo.ui.showInfoDialog({
                            message: '保存成功！'
                        });

                        var formatGroup = groupData.toJSON();
                        for(var k in data.${extUrl}){

                            groupData.set(k,data.projectExt[k]);
                        }
                    }else{
                        kendo.ui.showErrorDialog({
                            message: '保存失败!'
                        });
                    }
                }
            });
        },
        //清空该form，form不支持删除，提供清空选项，抹掉所有数据
        GroupFromClear:function (formKey) {

            var groupData = this[formKey];

            var col = "ext";
            for(var i = 1; i < 51; i++){

                groupData.set(col+i, null);
            }
        }

    });



</script>

<div id="tabstrip-groups" style="margin-left:5px;">
        <#-- 遍历，tab页头 -->
        <ul class="nav nav-tabs">
            <#list attributeGroups as group>

                <li <#if group_index == 0> class="k-state-active"</#if> >${group.groupDisplayName}</li>

            </#list>

        </ul>

            <#list attributeGroups as group>

                <div style="padding: 0px">
                    <#--表单形式的展示-->
                    <#if group.displayType == "FORM">

                    <div class="panel panel-default" style="border: 0px;margin-bottom: 0px">

                        <div class="panel-heading" style="padding: 4px">

                            <button style="float: right;margin-left: 5px" type="button" onclick="groupModel.GroupFromClear('${group.groupInternalName}${group.attributeGroupId}')" class="btn btn-warning">
                                <i class="fa  fa-eraser" style="margin-right:3px;"></i>清空</button>
                            <button style="float: right" type="button" onclick="groupModel.GroupFromSync('${group.groupInternalName}${group.attributeGroupId}')" class="btn btn-success">
                                <i class="fa fa-save" style="margin-right:3px;"></i>保存</button>

                        </div>

                        <div class="panel-body">

                            <form class="form-horizontal"
                                  id="${group.groupInternalName}${group.attributeGroupId}"
                                  role="form">
                                <div class="row" style="margin: 0px" >
                                <#-- 直接获取id本身没有任何问题，但是隐式的number转string的时候，freemark会自动加 “，”
                                例如： 123345 =》 “123,345” ，这个隐式的问题会导致key错误，从而导致找不到key而报错，所以强制调用 c 方法转换-->
                                    <#assign cols = groupsCols[group.attributeGroupId?c]>
                                    <#if (cols?size>0) >
                                        <#list cols as col>
                                            <#if col_index%4==0>
                                            <div class="col-md-12">
                                            </#if>
                                            <div class="col-md-3">
                                                <div class="form-group">
                                                    <label class="col-md-3 control-label" style="padding: 0px">${col.colDisplayName}</label>
                                                    <div class="col-md-9">
                                                        <#if col.formatCode == "BOOLEAN">
                                                            <input  type="checkbox"
                                                                    <#if col.requiredFlag?? && col.requiredFlag == "Y">
                                                                        required
                                                                    </#if>
                                                                    data-bind="value:${group.groupInternalName}${group.attributeGroupId}.${col.colInternalName}"
                                                                    id="${group.groupInternalName}_${col.colInternalName}"/>
                                                            <script>
                                                                $("#${group.groupInternalName}_${col.colInternalName}").kendoCheckbox({
                                                                            checkedValue:'Y',
                                                                            uncheckedValue:'N'
                                                                });
                                                            </script>
                                                        <#elseif col.formatCode == "DATE">
                                                            <input  type="text"
                                                                <#if col.requiredFlag?? && col.requiredFlag == "Y">
                                                                    required
                                                                </#if>
                                                                    data-bind="value:${group.groupInternalName}${group.attributeGroupId}.${col.colInternalName}"
                                                                    id="${group.groupInternalName}_${col.colInternalName}"
                                                                    style="width: 100%;"/>
                                                            <script>
                                                                $("#${group.groupInternalName}_${col.colInternalName}").kendoDatePicker({
                                                                    <#if col.formatMask??>
                                                                        format:"${col.formatMask}"
                                                                    </#if>
                                                                });
                                                                    <#if col.defaultValue??>
                                                                        groupModel.set('${group.groupInternalName}${group.attributeGroupId}.${col.colInternalName}','${col.defaultValue}');
                                                                    </#if>
                                                            </script>
                                                        <#elseif col.formatCode == "DATETIME">
                                                            <input  type="text"
                                                                <#if col.requiredFlag?? && col.requiredFlag == "Y">
                                                                    required
                                                                </#if>
                                                                    data-bind="value:${group.groupInternalName}${group.attributeGroupId}.${col.colInternalName}"
                                                                    id="${group.groupInternalName}_${col.colInternalName}"
                                                                    style="width: 100%;"/>
                                                            <script>
                                                                $("#${group.groupInternalName}_${col.colInternalName}").kendoDateTimePicker({
                                                                    <#if col.formatMask??>
                                                                        format:"${col.formatMask}"
                                                                    </#if>
                                                                });
                                                                    <#if col.defaultValue??>
                                                                        groupModel.set('${group.groupInternalName}${group.attributeGroupId}.${col.colInternalName}','${col.defaultValue}');
                                                                    </#if>
                                                            </script>
                                                        <#elseif col.formatCode == "NUM">
                                                            <input  type="text"
                                                                    <#if col.requiredFlag?? && col.requiredFlag == "Y">
                                                                        required
                                                                    </#if>
                                                                    data-bind="value:${group.groupInternalName}${group.attributeGroupId}.${col.colInternalName}"
                                                                    id="${group.groupInternalName}_${col.colInternalName}"
                                                                    style="width: 100%;"/>
                                                            <script>
                                                                $("#${group.groupInternalName}_${col.colInternalName}").kendoNumericTextBox({
                                                                    <#if col.formatMask??>
                                                                        format:"${col.formatMask}"
                                                                    </#if>
                                                                });
                                                                <#if col.defaultValue??>
                                                                    groupModel.set('${group.groupInternalName}${group.attributeGroupId}.${col.colInternalName}','${col.defaultValue}');
                                                                </#if>
                                                            </script>
                                                        <#elseif col.formatCode == "LIST">
                                                            <script src='${base.contextPath}/common/code?${group.groupInternalName}_${col.colInternalName+ "Resource"}=${col.formatValue}' type="text/javascript"></script>
                                                            <select data-role="combobox"
                                                                    id="${group.groupInternalName}_${col.colInternalName}"
                                                                    <#if col.requiredFlag?? && col.requiredFlag == "Y">
                                                                        required
                                                                    </#if>
                                                                    data-value-primitive="true"
                                                                    style="width:100%;"
                                                                    data-text-field="meaning"
                                                                    data-value-field="value"
                                                                    data-bind="value:${group.groupInternalName}${group.attributeGroupId}.${col.colInternalName}"
                                                                    data-source="${col.formatValue+ "Resource"}">
                                                            </select>
                                                            <script>
                                                                <#if col.defaultValue??>
                                                                    groupModel.set('${group.groupInternalName}${group.attributeGroupId}.${col.colInternalName}','${col.defaultValue}');
                                                                </#if>
                                                            </script>
                                                        <#elseif col.formatCode == "LOV">
                                                            <input
                                                                id="${group.groupInternalName}_${col.colInternalName}"
                                                                <#if col.requiredFlag?? && col.requiredFlag == "Y">
                                                                required
                                                                </#if>
                                                                type="text"
                                                                data-bind="value:${group.groupInternalName}${group.attributeGroupId}.${col.colInternalName}"
                                                                style="width: 100%;"/>

                                                            </input>
                                                            <script>
                                                                <#if lovDatas?? && col.formatValue??>
                                                                    <#assign lovData = lovDatas[col.formatValue]>
                                                                    <#if lovData??>
                                                                    $("#${group.groupInternalName}_${col.colInternalName}").kendoLov($.extend(${lovData},
                                                                            {
                                                                                select: function(e) {
                                                                                    /*viewModel.model.set('unitId', e.item.unitId);
                                                                                    viewModel.model.set('unitName', e.item.name);*/
                                                                                }
                                                                            }));
                                                                    </#if>
                                                                </#if>
                                                            </script>
                                                        <#else>
                                                            <input  type="text"
                                                                    data-bind="value:${group.groupInternalName}${group.attributeGroupId}.${col.colInternalName}"
                                                                    <#if col.requiredFlag?? && col.requiredFlag == "Y">
                                                                        required
                                                                    </#if>
                                                                    id="${group.groupInternalName}_${col.colInternalName}"
                                                                    class="k-textbox"
                                                                    style="width: 100%;"/>
                                                            <script>
                                                                <#if col.defaultValue??>
                                                                    groupModel.set('${group.groupInternalName}${group.attributeGroupId}.${col.colInternalName}','${col.defaultValue}');
                                                                </#if>

                                                            </script>

                                                        </#if>

                                                    </div>
                                                </div>
                                            </div>
                                            <#if (col_index+1)%4==0>
                                            </div>
                                            </#if>
                                        </#list>
                                    </div>
                                    <#else>
                                        <p style="margin: 10px;font-size: 17px">未配置属性列！</p>
                                    </#if>


                                </div>
                            </form>
                        </div>

                    </div>



                    <#--表格形式-->
                    <#elseif group.displayType == "GRID">
                        <#assign cols = groupsCols[group.attributeGroupId?c]>
                        <#if (cols?size>0) >
                            <div id='${"grid" + group_index}' style="border: 1px;"></div>
                            <script>
                                var ${"dataSource" + group_index} = new kendo.data.DataSource({
                                    transport: {
                                        read:  {
                                            url: "${base.contextPath}/attr/${extUrl}/gridQuery",
                                            type : "POST",
                                            dataType: "json"
                                        },
                                        update: {
                                            url: "${base.contextPath}/attr/${extUrl}/gridSubmit",
                                            contentType: "application/json",
                                            type: "POST"
                                        },
                                        create: {
                                            url: "${base.contextPath}/attr/${extUrl}/gridSubmit",
                                            contentType: "application/json",
                                            type: "POST"
                                        },
                                        destroy: {
                                            url: "${base.contextPath}/attr/${extUrl}/gridRemove",
                                            contentType: "application/json",
                                            type: "POST"
                                        },
                                        parameterMap: function(options, type) {
                                            if (type !== "read" && options.models) {
                                                var datas = Hap.prepareSubmitParameter(options, type);
                                                for(var i = 0,len = datas.length; i < len ; i++){
                                                    datas[i].primaryKey = viewModel.model.${pkName};
                                                    datas[i].${pkName} = viewModel.model.${pkName};
                                                }
                                                return kendo.stringify(datas);
                                            } else if (type === "read") {
                                                return Hap.prepareQueryParameter(
                                                        {
                                                            primaryKey:viewModel.model.${pkName},
                                                            <#if primaryKey??>
                                                                ${pkName}:viewModel.model.${pkName},
                                                            <#else>
                                                                ${pkName}:viewModel.model.${pkName},
                                                            </#if>
                                                            attributeGroupId:${group.attributeGroupId},
                                                            pkName:"${pkName}",
                                                            dtoFullName :"${dtoFullName}"

                                                        }, options);
                                            }
                                        }
                                    },
                                    requestEnd:function(e){
                                        if((e.type === "create" ||  e.type === "update") && e.response.success === true){
                                            kendo.ui.showInfoDialog({
                                                message:"保存成功！"
                                            })
                                        }
                                    },
                                    batch: true,
                                    pageSize: 10,
                                    schema: {
                                        data:'rows',
                                        total:'total',
                                        model: {
                                            id: "extId",
                                            fields: {
                                <#-- 数据源的数据配置 -->
                                    <#list cols as col>
                                        ${col.colInternalName}:{
                                            field :"${col.colInternalName}",

                                            <#if col.formatCode == "BOOLEAN">
                                                type:"boolean",
                                                checkedValue: 'Y',
                                                uncheckedValue: 'N',
                                            <#elseif col.formatCode == "GEN">
                                                type:"string",
                                            <#elseif col.formatCode == "DATE">
                                                type:"date",
                                            <#elseif col.formatCode == "DATETIME">
                                                type:"date",
                                            <#elseif col.formatCode == "NUM">
                                                type:"number",
                                            </#if>

                                            <#if col.defaultValue??>
                                                    defaultValue: "${col.defaultValue}",
                                            </#if>
                                            <#if col.requiredFlag?? && col.requiredFlag == "Y">
                                                validation: { required: true}
                                            </#if>
                                        },
                                    </#list>
                                        primaryKey: {
                                            type:"number",
                                            field:"${pkName}"
                                        },
                                        dtoFullName: {
                                            type:"string",
                                            field:"dtoFullName",
                                            defaultValue: "${dtoFullName}"
                                        },
                                        pkName: {
                                            type:"string",
                                            field:"pkName",
                                            defaultValue: "${pkName}"
                                        },
                                        attributeGroupId: {
                                            type:"number",
                                            field:"attributeGroupId",
                                            defaultValue: ${group.attributeGroupId}
                                        }
                                    }
                                }
                                }
                                });

                                var ${"grid" + group_index} =$("#${"grid" + group_index}").kendoGrid({
                                    toolbar: [
                                        {
                                            template:'<span style="float: right" class="btn btn-warning"  data-bind="click:${"grid" + group_index + "Cancel"}">' +
                                            '<i class="fa fa-warning"></i>取消</span>'

                                        },
                                        {
                                            template:'<span style="float: right" class="btn btn-success"  data-bind="click:${"grid" + group_index + "Save"}">' +
                                            '<i class="fa fa-save" style="margin-right:3px;"></i>保存</span>'

                                        },
                                        {
                                            template:'<span style="float: right" class="btn btn-primary"  data-bind="click:${"grid" + group_index + "Insert"}">' +
                                            '<i class="fa fa-plus-square" style="margin-right:3px;"></i>新增</span>'

                                        }
                                    ],
                                    dataSource: ${"dataSource" + group_index},
                                    navigatable: false,
                                    height:'100%',
                                    resizable: true,
                                    scrollable: true,
                                    editable:true,
                                    height:350,
                                    selectable:'row',
                                    rownumber: true,
                                    edit:function(e){

                                        if(typeof(viewModel.model.${pkName}) === "undefined"){
                                            kendo.ui.showInfoDialog({
                                                message: '请保存先保存项目，再进行属性组操作!'
                                            });
                                            preventDefault();
                                            return;
                                        }

                                        if(viewModel.model.${pkName} == null){
                                            kendo.ui.showInfoDialog({
                                                message: '请保存先保存项目，再进行属性组操作!'
                                            });
                                            preventDefault();
                                            return;
                                        }

                                    },
                                    pageable   : {
                                        pageSizes  : [5, 10, 20, 50],
                                        refresh    : true,
                                        buttonCount: 5
                                    },
                                    columns: [
                                        <#list cols as col>
                                            <#if !col.displayFlag?? || col.displayFlag =="Y" >
                                                {
                                                    field: "${col.colInternalName}",
                                                    title: '${col.colDisplayName}',
                                                    <#if col.formatMask??>
                                                        format:'{0:${col.formatMask}}',
                                                    </#if>
                                                    <#-- 当类型属于Boolean的时候，checkbox需要居中 -->
                                                    <#if col.formatCode == "BOOLEAN">
                                                        width:100,
                                                        attributes:{
                                                            style: "text-align: center;"
                                                        }
                                                    <#elseif col.formatCode == "LIST">
                                                        template: function (dataItem) {
                                                            var v = dataItem.${col.colInternalName};
                                                            $.each(${col.formatValue+ "Resource"}, function (i, n) {
                                                                if ((n.value || '')
                                                                        == (v || '')) {
                                                                    v = n.meaning;
                                                                    return v;
                                                                }
                                                            });
                                                            return v;
                                                        },
                                                        editor: function (container, options) {
                                                        $('<input  name="' + options.field + '"/>')
                                                                .appendTo(container)
                                                                .kendoDropDownList({
                                                                    dataTextField: "meaning",
                                                                    dataValueField: "value",
                                                                    dataSource: ${col.formatValue+ "Resource"}
                                                                });
                                                        }
                                                    <#elseif col.formatCode == "LOV">
                                                        template: function (dataItem) {
                                                            return dataItem['${col.colInternalName}'] || '';
                                                        },
                                                        editor: function(container, options){
                                                            <#if lovDatas?? && col.formatValue??>
                                                                <#assign lovData = lovDatas[col.formatValue]>
                                                                <#if lovData??>
                                                                    $('<input name="${col.colInternalName}" class="k-input k-text-box"/>')
                                                                            .appendTo(container)
                                                                            .kendoLov($.extend(${lovData}, {
                                                                                textField: '${col.colInternalName}',
                                                                                select:function(e){
                                                                                },
                                                                                model: options.model
                                                                            }));
                                                                </#if>
                                                            </#if>
                                                        }
                                                    </#if>
                                                },
                                            </#if>
                                        </#list>
                                        {
                                            title: '删除',
                                            width : 50,
                                            attributes  : {style: "text-align:center"},
                                            headerAttributes: {
                                                style  : "text-align: center"
                                            },
                                            command:[{
                                                name:"destroy",
                                                text:""
                                            }]
                                        }

                                    ]
                                }).data("kendoGrid");

                            </script>

                        <#else >
                            <p style="margin: 10px;font-size: 17px">未配置属性列！</p>

                        </#if>
                    </#if>
                </div>
            </#list>

</div>

<script>

    $("#tabstrip-groups").kendoTabStrip({
    });


    <#-- bind groupModel -->
    kendo.bind($('#tabstrip-groups'), groupModel);

    <#-- 属性组form数据的绑定， 读取，更新和新建操作 -->

    groupModel.groupFromsRead();

</script>
<#else>
<div>

</div>
</#if>
```