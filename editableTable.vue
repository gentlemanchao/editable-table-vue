<template>
    <div :id="id" class="c-editable-table" :style="{width:tableWidth,height:tableHeight}">
        <div class="c-editable-table-thead">
            <table width="100%" cellspacing="0" cellpadding="0" border="0">
                <colgroup>
                    <col v-for="item in columns" :key="item.key" :width="item.width">
                </colgroup>
                <thead>
                    <tr>
                        <th v-for="item in columns" :key="item.key">
                            <div class="c-editable-table-cell" :style="{textAlign:!item.text?'center' : 'left'}">
                                <span>{{item.title}}</span>
                            </div>
                        </th>
                    </tr>
                </thead>
            </table>
        </div>
        <div class="c-editable-table-tbody" :style="{height:tbodyHeight}">
            <table width="100%">
                <colgroup>
                    <col v-for="item in columns" :key="item.key" :width="item.width">
                </colgroup>
                <tbody>
                    <tr v-for="(list,index) in data" :key="index +'-'+ (specialKey ? list[specialKey]:'')">
                        <td v-for="item in columns" :key="item.key">
                            <div v-if="item.renderBtns" class="c-editable-table-cell">
                                <a href="javascript:void(0)" class="c-editable-table-cell-btn" v-for="btn in item.renderBtns" :key="btn.key"  :class="{' c-editable-table-cell-btn-edit':btn.isEdit}" @click="_onTableCellBtnClick({row:list,index:index},btn,arguments[0])">
                                    {{(typeof(list['isOnEdit']) !== 'undefined' && list['isOnEdit'] === true && btn.isEdit) ? '保存' : btn.name}}
                                </a>
                            </div>
                            <template v-else-if="item.text">
                                <div v-if="typeof(list['isOnEdit'])!== 'undefined' && list['isOnEdit'] === true" class="c-editable-table-cell c-cell-editable c-table-cell-input">
                                    <input class="c-editable-table-input" type="text" :value="list[item.key]">
                                </div>
                                <div v-else class="c-editable-table-cell c-cell-editable">
                                    <span v-if="item.renderText">{{item.renderText(list[item.key])}}</span>
                                    <span v-else>{{list[item.key]}}</span>
                                </div>
                            </template>
                        </td>
                    </tr>

                </tbody>
            </table>
        </div>
    </div>
</template>
<script>
import $ from 'jQuery';
import './styles/editableTable.style';
import Util from './scripts/util';
export default {
    data() {
        return {
            id: 'editableTable',
        }
    },
    computed: {
        tableWidth() {
            return typeof (this.width) == "number" ? this.width + 'px' : this.width;
        },
        tableHeight() {
            return typeof (this.height) == "number" ? this.height + 'px' : this.height;
        },
        tbodyHeight() {
            return typeof (this.height) == "number" ? this.height - 40 + 'px' : "auto";
        }

    },
    props: {
        width: {
            type: [Number, String],
            default: 800
        },
        height: {
            type: [Number, String],
            default: 300
        },
        specialKey:{//独一无二的key
            type:String
        },
        columns: {
            type: Array,
            default: () => {
                return [
                    {
                        title: '名称',
                        key: 'name',
                        width: "100px",
                        text: true
                    },
                    {
                        title: '取值',
                        key: 'title',
                        text: true,
                        renderText: (text) => {//字符串编辑
                            return text + 'change like this';
                        }
                    },
                    {
                        title: '描述',
                        key: 'describle',
                        text: true
                    },
                    {
                        title: '操作',
                        key: 'commands',
                        width: "300px",
                        text: false,
                        renderBtns: [
                            {
                                isEdit: true,
                                name: "编辑",
                                key: "edit",
                                func: (data) => {
                                    console.log("edit")
                                    console.log(data)
                                }
                            },
                            {
                                isEdit: false,
                                name: "删除",
                                key: "delete",
                                func: (data) => {
                                    console.log("delete")
                                    console.log(data)
                                }
                            }
                        ]
                    }
                ]
            }
        },
        data: {
            type: Array,
            default: () => {
                return [
                    {
                        name: "名称",
                        title: "取值",
                        describle: "描述",
                        id: "id_000001"
                    },
                    {
                        name: "名称2",
                        title: "取值2",
                        describle: "描述2",
                        id: "id_000002"
                    },
                ]
            }
        }
    },

    watch:{
        data(val,old){
            console.log('--------------------')
            console.log(val);
        
        }
    },
    created() {
        this.id = 'editableTable_' + new Date().getTime();
    },
    mounted() {
    },
    beforeDestroyed() {
    },
    methods: {
        //获取输入框数据
        _getInputData(el){
           let $btn = $(el);
            let $children = $btn.parents('tr').find('.c-editable-table-cell.c-table-cell-input');
            let columnList = getColumnTextList(this.columns);
            return _getInputData($children, columnList);
        },
        //设置表格项为输入框
        _setCellToInput(el) {
            let $btn = $(el);
            let $children = $btn.parents('tr').find('.c-editable-table-cell.c-cell-editable');
            changeCellToInput($children);
            $btn.text("保存");
        },
        //设置输入框为表格
        _setInputToCell(el){
            let $btn = $(el);
            let $children = $btn.parents('tr').find('.c-editable-table-cell.c-table-cell-input');
            changeInputToCell($children);
            $btn.text("编辑");
        },
        /***
         * 按钮点击事件用于分发到对应按钮的func方法
         *  @param {obj} param 当前行数据
         *  @param {obj} btn 当前按钮属性
         *  @param {obj} e 点击事件对象
         */
        _onTableCellBtnClick(param,btn,e) {
            let data = param.row;
            let func = btn.func;
            if (btn.isEdit && btn.isEdit == true) {//编辑按钮
                if (typeof (data['isOnEdit']) == "undefined" || !data['isOnEdit']) {
                    data['isOnEdit'] = true;
                    this._setCellToInput(e.target);
                } else {
                    let _data = this._getInputData(e.target);
                    let result = Object.assign(data, _data);
                    param.row = result;
                    if(func(param)){
                        data['isOnEdit'] = false;
                        this._setInputToCell(e.target);
                    }
                }
            } else {
                func(param);
            }
        },
        /**
         * 当前行数据保存成功回调方法
         * @param {integer} index 当前数据的索引 param.index
         */
        editSucceed(index){
            let tr = $('#' + this.id + ' .c-editable-table-tbody tr')[index];
            let $children = $(tr).find('.c-editable-table-cell.c-table-cell-input');
            changeInputToCell($children);
            let $btn = $(tr).find('.c-editable-table-cell-btn-edit');
            $btn.text("编辑");
            this.data[index]['isOnEdit'] = false;
        },
        /**
         *设置表格行为输入框
         * @param
         * */
        setCellToInput(index){
            let tr = $('#' + this.id + ' .c-editable-table-tbody tr')[index];
            let $children = $(tr).find('.c-editable-table-cell.c-cell-editable');
            changeCellToInput($children);
            let $btn = $(tr).find('.c-editable-table-cell-btn-edit');
            $btn.text("保存");
            this.data[index]['isOnEdit'] = true;
        }
    }
}

//获取表格数据项显示字段列表
let getColumnTextList = (columns) => {
    let i, d, arr = [];
    for (i = 0; i < columns.length; i++) {
        d = columns[i];
        if (d.text && d.text === true) {
            arr.push(d.key);
        }
    }
    return arr;
};

//提取编辑中的数据
let _getInputData = ($domList, columnList)=>{
    let i, dom, $dom, data, str, result = {};
    for (i = 0; i < $domList.length; i++) {
        dom = $domList[i];
        $dom = $(dom);
        if (!$dom.hasClass("c-table-cell-input")) continue;
        data = $dom.children("input").val();
        result[columnList[i]] = data;
    }
    return result;
};

//输入框转表格
let changeInputToCell = ($domList)=>{
    let i, dom, $dom, data, str, result = {};
    for (i = 0; i < $domList.length; i++) {
        dom = $domList[i];
        $dom = $(dom);
        if (!$dom.hasClass("c-table-cell-input")) continue;
        data = $dom.children("input").val();
        str = '<span>' + data + '</span>';
        $dom.html(str).removeClass('c-table-cell-input')
    }
};

//表格转输入框
let changeCellToInput = ($domList) => {
    $domList.each(function() {
        let $this = $(this), data, str;
        if ($this.hasClass("c-table-cell-input")) { return; }
        data = $this.children("span").text();
        str = '<input class="c-editable-table-input" type="text" value="' + data + '">';
        $this.html(str).addClass('c-table-cell-input')
    });
};
</script>
