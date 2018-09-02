<template>
  <div class="select-bar" style="width: width">
    <div class="chapter" v-for="(item, index) in data[defaultProps.chapter]" :key="item.id" :style="{'margin-top': chapterMarginTop + 'px'}">
      <!-- chapter-mark是为了实现章的拖动 -->
      <div class="chapter-item"  @click="handleChapter($event)"
        :style="{'background-color': color.chapterBackground, 'color': color.chapterColor}">
        <i class="el-icon-caret-right mark"></i>
        <span>{{ item.name }}</span>
        <input type="text" class="order-num" :value="item[defaultProps.orderNum]" v-if="isShowChapterOrder" @click.stop.prevent="chapterInputClick"
          @focus.stop.prevent="chapterFocus(item[defaultProps.orderNum])" @blur.stop.prevent="chapterBlur(item, $event)" :maxlength="maxLength">
        <div class="icon-chapter-group">
          <!-- 这儿是父集的icon -->
          <i class="el-icon-circle-plus-outline" @click.stop.prevent="addSection(item)"></i>
          <i class="el-icon-edit" @click.stop.prevent="editChapter(item)"></i>
          <i class="el-icon-delete" @click.stop.prevent="deleteChapter(item)"></i>
        </div>
      </div>
      <div class="section-wrapper" v-if="item[defaultProps.children].length>0">
        <!-- 通过section-box设置列表显示的高度，并且实现动画 -->
        <div class="section-box">
          <!-- 这儿section-mark的目的是应用Sortable，实现section的拖动，而且必须设置高度（不大于一个section的高度）,
          否则，当其中一个拖完了以后，无法再向里面拖动新的，-->
          <div class="section-mark" style="height: 5px" :data-index="index" >
            <div class="section" v-for="child in item[defaultProps.children]" :key="child.id" @click.stop.prevent="editSectionTemp(child)"
              :style="{'background-color': color.sectionBackground, 'color': color.sectionColor, 'margin-top': sectionMarginTop + 'px'}">
                <span>{{ child[defaultProps.orderNum] }}</span>
                <span>{{ child.name }}</span>
                <input type="text" class="section-input" :value="child[defaultProps.orderNum]" v-if="isShowSectionOrder" :maxlength="maxLength"
                @focus.stop.prevent="sectionFocus(child[defaultProps.orderNum])" @blur.stop.prevent="sectionBlur(child, $event)">
                <div class="icon-section-group">
                  <!-- 这儿是子集的icon -->
                  <i class="el-icon-edit" @click.stop.prevent="editSection(child)"></i>
                  <i class="el-icon-delete" @click.stop.prevent="deleteSection(child)"></i>              
                </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import Sortable from 'sortablejs';
export default {
  data () {
    return {
      mark: true,
      chapterInputValue: 0,
      sectionInputValue: 0
    };
  },
  props: {
    data: {
      type: Object
    },
    // 是否排序
    sort: {
      type: Boolean,
      default: false
    },
    // 根据实际情况对应的属性
    defaultProps: {
      type: Object,
      default: function () {
        return {
          chapter: 'chapter',
          children: 'children',
          orderNum: 'orderNum'
        }
      }
    },
    color: {
      type: Object,
      default: {
        chapterBackground: '#E6ECF0',
        sectionBackground: '#F4F7F9',
        chapterColor: '#656565',
        sectionColor: '#999999'
      }
    },
    width: {
      type: String,
      default: '100%'
    },
    // 是否每次只展开一个, 默认为false
    accordion: {
      type: Boolean,
      default: false
    },
    // 单个小节的margin-top值
    sectionMarginTop: {
      type: Number,
      default: 10
    },
    // 章的margin-top值
    chapterMarginTop: {
      type: Number,
      default: 10
    },
    // 小节是否可以拖动
    isSectionSortable: {
      type: Boolean,
      default: true
    },
    // 章是否可以拖动
    isChaterSortable: {
      type: Boolean,
      default: true
    },
    // 是否可以通过点击小节本身触发编辑事件
    isSectionClickToEdit: {
      type: Boolean,
      default: false
    },
    // 是否显示chapter手动排序输入框
    isShowChapterOrder: {
      type: Boolean,
      default: true
    },
    // 是否显示section手动排序输入框
    isShowSectionOrder: {
      type: Boolean,
      default: true
    },
    // 可以输入最大长度
    maxLength: {
      type: Number,
      default: 2
    }
  },
  mounted () {
    let _this = this;
    setTimeout(() => {
      var sectionWrapperAll = document.querySelectorAll('.section-mark');
      var chapter = document.querySelector('.select-bar');
      if (this.isSectionSortable) {
        var arr = [];
        sectionWrapperAll.forEach((ele, index) => {
          arr[index] = Sortable.create(ele, {
            // 通过group设置是否可以拖出去或者拖进来
            group: {name: 'name', pull: false, put: false},
            sort: true,
            dataIdAttr: 'order',
            disabled: false,
            animation: 200,
            forceFallback: false,
            onEnd (evt) {
              // evt.to是移动到的列表容器
              evt.to.parentNode.style.height = evt.to.scrollHeight + this.sectionMarginTop + 'px';
              evt.target.parentNode.style.height = evt.target.scrollHeight + this.sectionMarginTop + 'px';

              console.log('这儿移动小节');
              console.log(evt.oldIndex);
              console.log(evt.newIndex);
              let index = +evt.to.dataset.index;
              console.log(index);

              console.log('key');
              let key = _this.defaultProps.chapter;

              console.log('这儿是子对象的key');
              let childKey = _this.defaultProps.children;
              
              console.log('parent');
              console.log(_this.data[key]);
              // evt.oldIndex 节点的老位置
              // evt.newIndex 节点的新位置
              _this.$emit('sectionIndexChange', evt.newIndex, _this.data[key][index][childKey][evt.oldIndex]);
            },
          });
        });
      }
      if (this.isChaterSortable) {
        var sortable = Sortable.create(chapter, {
          // 通过group设置是否可以拖出去或者拖进来
          group: {name: 'name', pull: false, put: false},
          sort: true,
          dataIdAttr: 'order',
          disabled: false,
          animation: 200,
          forceFallback: false,
          onEnd (evt) {
            // evt.to是移动到的列表容器
            evt.to.parentNode.style.height = evt.to.scrollHeight + this.sectionMarginTop + 'px';
            evt.target.parentNode.style.height = evt.target.scrollHeight + this.sectionMarginTop + 'px';
            // evt.oldIndex 节点的老位置
            // evt.newIndex 节点的新位置
            let key = _this.defaultProps.chapter;
            _this.$emit('chapterIndexChange', evt.newIndex, _this.data[key][evt.oldIndex]);
          },
        });
      }
    }, 20)
  },
  methods: {
    // 当章被点击时
    handleChapter (event) {
      var iconMark = event.currentTarget.querySelector('.mark');
      // 控制小节列show 和 hide的盒子
      var sectionBox = event.currentTarget.parentNode.querySelector('.section-box');
      // 如果每次只展开一个为true
      if (this.accordion) {
        // 如果被点击的章是展开的，name就收起
        if (iconMark.classList.contains('el-icon-caret-bottom')) {
          iconMark.classList.add('el-icon-caret-right');
          iconMark.classList.remove('el-icon-caret-bottom');
          // 将sectionBox的高度设置为0
          sectionBox.style.height = 0;
          return;
        } else {
          // 如果点击的章是收起的，那么首先找到其他的展开的章，进行关闭，然后再将自己这章打开
          var chapterItem = this.$el.querySelector('.el-icon-caret-bottom');
          if (chapterItem) {
            chapterItem.classList.remove('el-icon-caret-bottom');
            chapterItem.classList.add('el-icon-caret-right');
          }
          iconMark.classList.remove('el-icon-caret-right');
          iconMark.classList.add('el-icon-caret-bottom');
          var sectionBoxOpen = document.querySelector('.showSection');
          if (sectionBoxOpen) {
            sectionBoxOpen.style.height = 0;
            sectionBoxOpen.classList.remove('showSection');
          }
          sectionBox.style.height = sectionBox.scrollHeight +'px';
          sectionBox.classList.add('showSection');
        }
      } else {
        if (iconMark.classList.contains('el-icon-caret-bottom')) {
          // 如果每次只展开一个设置为false          
          iconMark.classList.add('el-icon-caret-right');
          iconMark.classList.remove('el-icon-caret-bottom');
          // 将sectionBox的高度设置为0
          sectionBox.style.height = 0;
          return;
        } else {
          iconMark.classList.add('el-icon-caret-bottom');
          iconMark.classList.remove('el-icon-caret-right');
          sectionBox.style.height = sectionBox.scrollHeight + 'px';
        }
      }
    },
    // 添加小节
    addSection (item) {
      this.$emit('addSection', item);
    },
    // 编辑章
    editChapter (item) {
      this.$emit('editChapter', item);
    },
    // 删除章
    deleteChapter (item) {
      this.$emit('deleteChapter', item);
    },
    // 编辑小节
    editSection (item) {
      this.$emit('editSection', item);
    },
    // 点击的小节本身
    editSectionTemp (item) {
      if (this.isSectionClickToEdit) {
        this.editSection(item);
      }
    },
    // 删除小节
    deleteSection (item) {
      this.$emit('deleteSection', item);
    },
    // 章的排序获取焦点
    chapterFocus (value) {
      this.chapterInputValue = value;
    },
    // 章的排序失去焦点
    chapterBlur (item, event) {
      // console.log(event.target.value);
      if (!event.target.value) {
        this.$message.error('排序不能为空');
        event.target.value = this.chapterInputValue;
        return;
      }
      if (!this.isNumber(event.target.value)) {
        this.$message.error('请输入正确的排序');
        event.target.value = this.chapterInputValue;
        return;
      }
      this.$emit('chapterBlurValue', event.target.value, item);
    },
    // 小节的排序获取焦点
    sectionFocus (value) {
      this.sectionInputValue = value;
    },
    sectionBlur (item, event) {
      if (!event.target.value) {
        this.$message.error('排序不能为空');
        event.target.value = this.sectionInputValue;
        return;
      }
      if (!this.isNumber(event.target.value)) {
        this.$message.error('请输入正确的排序');
        event.target.value = this.sectionInputValue;
        return;
      }
      this.$emit('sectionBlurValue', event.target.value, item);
    },
    chapterInputClick () {
    },
    // 通过正则判断输入的是否为正确的内容
    isNumber (number) {
      console.log('这儿是number')
      console.log(number)
      let reg = /^[1-9]\d{1,5}$|^\d$/;
      console.log(reg.test(number));
      return reg.test(number);
    }
  }
};
</script>

<style rel="stylesheet/scss" lang="scss" scoped>
.select-bar {
  .chapter {
    i {
      font-size: 18px;
    }
    .chapter-item {
      position: relative;
      border-radius: 3px;
      cursor: pointer;
      padding: 0 20px 0 10px;
      height: 40px;
      line-height: 40px;
      overflow: hidden;
      .icon-chapter-group {
        float: right;
        i {
          margin-left: 8px;
        }
      }
      .order-num {
        position: absolute;
        right: 115px;
        top: 5px;
        width: 40px;
        border: none;
        height: 25px;
        outline-style: none;
        text-align: center;
        border-radius: 3px;
        border: solid 1px #fff;
        color: #6e6e6e;
        &:focus {
          border-color: #069;
        }
      }
    }
    .section-wrapper {
      .section-box {
        overflow: hidden;
        height: 0;
        transition: all .3s;
        .section {
          position: relative;
          margin: 0 0 0 15px;
          height: 30px;
          line-height: 30px;
          border-radius: 3px;
          padding: 0 20px 0 10px;
          cursor: pointer;
          .icon-section-group {
            float: right;
            i {
              cursor: pointer;
              margin-left: 8px;
            }
          }
          .section-input {
            position: absolute;
            right: 85px;
            top: 3px;
            width: 35px;
            border: none;
            height: 20px;
            outline-style: none;
            text-align: center;
            border-radius: 3px;
            border: solid 1px #fff;
            color: #6e6e6e;
            &:focus {
              border-color: #069;
            }
          }
        }
      }
    }
  }
}
</style>
