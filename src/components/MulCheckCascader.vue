<template lang='html'>
    <div class='vk-multil-cascader'>
        <el-popover placement="top-start" popper-class="vk-multi-cascader-popover" :visible-arrow="showArrow" trigger="click" @hide="whenPopoverHide" @show="whenPopoverShow">
            <muContent
                :height="height"
                :width="width"
                :option="options"
                @handleOutPut="whenOutPut"
                :selectedValues="selectedValues"
                :outputType="outputType"
                :disabledPair="disabledPair">
            </muContent>
            <el-input popper-class="slect-panel" v-if="activeItem[0] && activeItem[0].level === 0"  v-model="inputValue" readonly slot="reference" :suffix-icon="inputArrow">
            </el-input>
        </el-popover>
    </div>
</template>

<script>
import muContent from "./multiContent";
// 基于 vue.js && element-ui 的多选级联选择器
export default {
    name: "multiCascader",
    props: {
        options: {
            type: Array,
            default() {
                return [];
            }
        },
        width: {
            type: String,
            default: ""
        },
        height: {
            type: String,
            default: ""
        },
        inputValue: {
            type: String,
            default() {
                return "";
            }
        },
        // 输出值的类型
        outputType: {
            type: String,
            default() {
                return "value";
            }
        },
        disabledPair: {
            type: Object,
            default() {
                return {};
            }
        }
    },
    data() {
        return {
            selectedValues: [],
            showArrow: true,
            activeItem: [],
            outputValue: [],
            optionDicts: [],
            inputArrow: "el-icon-arrow-down",
            popoverWidth: "",
            // 展开之后的数组， 将每一个children 展开
            flatOptions: []
        };
    },
    watch: {
        "options": function () {
            this.initData();
        }
    },
    components: {
        muContent
    },
    created() {
        this.initData();
        this.setOptionDicts(this.options);
        this.toFlatOption(this.options);
    },
    methods: {
        whenPopoverHide() {
            this.inputArrow = "el-icon-arrow-down";
        },
        whenPopoverShow() {
            this.inputArrow = "el-icon-arrow-up";
        },
        initData() {
            this.setLevel();
            this.activeItem = this.options;
            const { width, height } = this;
            const checkedValues = [];
            let childrenValues = [];
            const getChecked = (item) => {
                const { checked, value, children, level, siblingValues } = item;
                if (siblingValues) {
                    const tempValues = [...siblingValues];
                    item.siblingValues = tempValues;
                }
                childrenValues.push(value);
                if (children && children.length > 0) {
                    children.forEach(child => {
                        getChecked(child);
                    });
                } else {
                    if (checked) checkedValues.push(item[this.outputType]);
                }
            };
            this.options.forEach(child => {
                getChecked(child);
                // 设置当前item 的 childrenValues, 包含当前item 下的所有值的 value
                child.childrenValues = [...childrenValues];
                childrenValues = [];
            });
            this.selectedValues = checkedValues;
            this.whenOutPut(this.selectedValues);
        },
        getTypeOptions(values, outputType) {
            const outputValues = [...values];
            const finalOutputArr = [];
            return this.flatOptions.reduce((pev, cur) => {
                const { value: curVal } = cur;
                if (outputType === "item") {
                    if (outputValues.includes(curVal)) pev.push(cur);
                } else {
                    if (outputValues.includes(curVal) && cur[outputType]) pev.push(cur[outputType]);
                }
                return pev;
            }, []);
        },
        toFlatOption(option) {
            const getItems = (arr, cur) => {
                const keys = Object.keys(cur);
                const newObj = {};
                keys.forEach(key => key !== "children" && (newObj[key] = cur[key]));
                arr.push(newObj);
                const curChild = cur.children;
                if (curChild && curChild.length > 0) {
                    return curChild.reduce((array, now) => getItems(array, now), arr);
                } else {
                    return arr;
                }
            };
            this.flatOptions = option.reduce((arr, cur) => getItems(arr, cur) && arr, []);
        },
        // 设置配置的字典
        setOptionDicts(options) {
            if (!Array.isArray(options)) {
                const { label, value } = options;
                this.optionDicts.push({ value, label });
                const children = options.children;
                if (children) {
                    this.setOptionDicts(children);
                }
            } else {
                options.forEach(opt => {
                    this.setOptionDicts(opt);
                });
            }
        },
        whenOutPut(value) {
            // 根据选中的值数组 value 输出特定 outputType 类型
            if (this.outputType !== "value") {
                this.outputValue = this.getTypeOptions(value, this.outputType);
            } else {
                this.outputValue = value;
            }
            this.$emit("on-selected", this.outputValue);
        },
        // 设定层级
        setLevel() {
            const siblingValues = [];
            let tempLevel = 0;
            if (this.options.length) {
                const addLevel = option => {
                    const optChild = option.children;
                    if (option.level === tempLevel) {
                        siblingValues.push(option.value);
                    }
                    if (optChild) {
                        optChild.forEach(opt => {
                            opt.level = option.level + 1;
                            addLevel(opt);
                        });
                    }
                };
                this.options.forEach(option => {
                    if (!option.level) {
                        option.level = 0;
                        tempLevel = option.level;
                    }
                    addLevel(option);
                    option.siblingValues = siblingValues;
                });
            }
        },
        showSecondLevel(item) {
            this.activeItem = item;
        }
    }
};
</script>
<style lang='less' scoped>
.vk-menu-item {
    display: flex;
    justify-content: space-between;
    align-items: center;
    list-style-type: none;
    white-space: nowrap;
    overflow: hidden;
    text-overflow: ellipsis;
    cursor: pointer;
    outline: none;
    padding: 8px 20px;
    font-size: 14px;
    width: 100%;
    &:hover {
        background-color: rgba(125,139,169,.1);
    }
}
@import "~assets/css/variable";
</style>
