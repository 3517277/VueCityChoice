<template>
  <div class="main_layout">
    <div class="input_layout" :class="{ expand: isexpand }">
      <a-tag
        color="#2db7f5"
        class="tag_item"
        :closable="multiple"
        v-for="item in checkedList"
        :key="item.name"
        @close.stop="tagClose(item)"
        >{{ item.name }}</a-tag
      >
      <a-input
        id="inputitem"
        v-model="searchText"
        class="input_item"
        @focus="cityexpand"
        @focusout="cityup"
        placeholder="请选择城市"
        @change="search"
      />
    </div>
    <div
      ref="tabs_layout"
      class="tabs_layout"
      tabindex="0"
      v-show="isexpand"
      @focusout="cityup"
    >
      <!--这里是搜索界面-->
      <template v-if="searching">
        <div class="search_panel">
          <template v-if="multiple">
            <div class="search_checkbox_panel">
              <a-checkbox
                class="search_checkbox_item"
                v-for="(item, tabIndex) in searchDatas"
                :key="tabIndex"
                :checked="item.checked"
                @change.stop="checkBoxChanged({ ...item, $event })"
                >{{ item.name }}</a-checkbox
              >
            </div>
          </template>
          <template v-else>
            <label
              class="search_label_item"
              v-for="(item, index) in this.searchDatas"
              :key="index"
              @click.stop="checkBoxChanged(item)">{{ item.name }}</label>
          </template>
        </div>
      </template>
      <!--这里是正常界面-->
      <template v-else>
        <a-tabs ref="tab" class="tab" :default-active-key="0">
          <a-tab-pane
            class="tab_panel"
            v-for="(item, tabIndex) in cityListKey"
            :key="tabIndex"
            :tab="item"
          >
            <div
              v-for="(item2, groupIndex) in tabDatas[tabIndex]"
              :key="groupIndex"
            >
              <div class="lable">{{ item2.ckey }}</div>
              <div class="checkbox_panel">
                <template v-if="multiple">
                  <a-checkbox
                    class="check_item"
                    v-for="(item3, index) in item2.cityList"
                    :key="index"
                    :checked="item3.checked"
                    @change.stop="checkBoxChanged({ tabIndex, groupIndex, index, $event })">{{ item3.name }}</a-checkbox
                  >
                </template>
                <template v-else>
                  <label
                    class="label_item"
                    v-for="(item3, index) in item2.cityList"
                    :key="index"
                    @click.stop="
                      checkBoxChanged({ tabIndex, groupIndex, index, $event })
                    "
                    >{{ item3.name }}</label
                  >
                </template>
              </div>
            </div>
          </a-tab-pane>
        </a-tabs>
      </template>
    </div>
  </div>
</template>

<script>
import citys from '@/assets/citys.json'
import untils from '@/untils/common.js'

export default {
  data () {
    return {
      isexpand: false,
      tabDatas: [], // 分组后的数据
      cityListKey: [], // "ABCD"Tabs标题
      checkedList: [], // 当前选定的checkBox
      searching: false, // 是否处于搜索状态
      searchDatas: [], // 查找到的数据
      searchText: '', // 查找内容
      allCityList: [] // 单个数据，对象内有它所在的下标路径
    }
  },
  props: {
    multiple: {
      // 多选
      type: Boolean,
      default: false
    },
    values: {
      // 双向绑定
      type: Array,
      default: function () {
        return []
      }
    }
  },
  model: {
    prop: 'values',
    event: 'citychanged'
  },
  created () {
    if (Object.entries(this.tabDatas).length === 0) {
      console.log('this.loadDatas()')
      this.loadDatas()
    }
  },
  mounted () {
    // 挂载完成时，如果需要初始化就进行初始化
    if (this.values.length > 0) {
      this.updateValues(this.values)
    }
  },
  watch: {
    values: function (newVal) {
      this.updateValues(newVal)
    }
  },
  methods: {
    cityexpand () {
      if (!this.isexpand) {
        this.isexpand = true

        if (Object.entries(this.tabDatas) === 0) {
          this.loadDatas()
        }
      }
    },
    cityup (e) {
      if (this.isexpand) {
        if (!untils.isParent(e.relatedTarget, this.$refs.tabs_layout)) {
          this.isexpand = false
        }
      }
    },
    loadDatas () {
      const cityDatas = {}
      citys.map(item => {
        const key = item.pinyin.charAt(0).toUpperCase() // 根据key值的第一个字母分组，并且转换成大写
        const dictValue = cityDatas[key] || [] // 如果tabDatas里面有这个key了，就取，没有就是空数组
        dictValue.push({
          // 增加数组内容
          label: item.label,
          name: item.name,
          checked: false
        })

        cityDatas[key] = dictValue
      })

      // 转成对象数组
      let list = []
      for (const gkey in cityDatas) {
        list.push({
          ckey: gkey,
          cityList: cityDatas[gkey]
        })
      }

      // 排序(根据ASCII码)
      list = list.sort(
        (item1, item2) => item1.ckey.charCodeAt(0) - item2.ckey.charCodeAt(0)
      )

      // 4个字母一组
      const chunk = 4
      for (var i = 0, j = list.length; i < j; i += chunk) {
        this.tabDatas.push(list.slice(i, i + chunk))
      }

      // 取得分组标题
      const cityListKey = []
      this.tabDatas.map(item => {
        let ckeys = ''
        item.map(childItem => {
          ckeys += childItem.ckey
        })

        cityListKey.push(ckeys)
      })

      this.cityListKey = cityListKey

      // 所有对象全部放在一个数组中，里面有下标的路径
      this.tabDatas.forEach((item, index) => {
        item.forEach((item2, index2) => {
          item2.cityList.forEach((item3, index3) => {
            const newItem = item3
            newItem.tabIndex = index
            newItem.groupIndex = index2
            newItem.index = index3
            this.allCityList.push(newItem)
          })
        })
      })
    },
    updateValues(newVal){ // 初始化数据
      // 去重
      const unique = []
      newVal.forEach(item => {
        if (unique.indexOf(item) === -1) {
          unique.push(item)
        }
      })

      // 先勾选掉不再存在的数组
      this.checkedList.forEach((item) => {
        if (unique.indexOf(item.name) !== -1) {
          const checkbox = this.tabDatas[item.tabIndex][item.groupIndex]
            .cityList[item.index]
          checkbox.checked = false
          // this.checkedList.splice(index, 1);
        }
      })

      this.checkedList = []

      unique.forEach(name => {
        const index = this.allCityList.findIndex(item => item.name === name)
        if (index !== -1) {
          const item = this.allCityList[index]
          // 更改选中状态
          this.changeCheckedStatus(
            item.tabIndex,
            item.groupIndex,
            item.index,
            true,
            true
          )
        }
      })
    },

    checkBoxChanged (e) {
      const { tabIndex, groupIndex, index, $event: event } = e

      this.changeCheckedStatus(
        tabIndex,
        groupIndex,
        index,
        this.multiple ? event.target.checked : true
      )

      if (!this.multiple) {
        this.isexpand = false
      }
    },

    tagClose (e) {
      const { tabIndex, groupIndex, index } = e
      this.changeCheckedStatus(tabIndex, groupIndex, index, false)
    },

    changeCheckedStatus (
      tabIndex,
      groupIndex,
      index,
      checked,
      forbidEmit = false
    ) {
      const item = this.tabDatas[tabIndex][groupIndex].cityList[index]
      item.checked = checked

      if (item.checked) {
        if (!this.multiple) {
          this.checkedList = []
        }

        this.checkedList.push({
          name: item.name,
          tabIndex,
          groupIndex,
          index
        })
      } else {
        const index = this.checkedList.findIndex(obj => obj.name === item.name)
        this.checkedList.splice(index, 1)
      }

      // this.$emit(
      //   "cityChanged",
      //   this.checkedList.map(item => item.name)
      // );

      if (!forbidEmit) {
        this.citychanged()
      }
    },
    citychanged () {
      this.$emit('citychanged', this.checkedList.map(item => item.name))
    },

    search() {
      if (this.searchText === '') {
        this.searching = false
      } else {
        this.searchDatas = []
        // const datas = []
        // this.tabDatas.forEach((item, index) => {
        //   item.forEach((item2, index2) => {
        //     item2.cityList.forEach((item3, index3) => {
        //       const newItem = item3
        //       newItem.tabIndex = index
        //       newItem.groupIndex = index2
        //       newItem.index = index3
        //       datas.push(newItem)
        //     })

        //     this.searchDatas = datas.filter(
        //       city => city.label.indexOf(this.searchText) !== -1
        //     )
        //   })
        // })

        this.searchDatas = this.allCityList.filter(
          city => city.label.indexOf(this.searchText) !== -1
        )

        this.searching = true
      }
    }
  }
}
</script>

<style lang="less">
.main_layout {
  position: relative;
  .input_layout {
    border: 1px solid #d9d9d9;
    border-radius: 4px;
    padding: 0 8px;
    width: 100%;
    display: flex;
    transition: all 0.3s;
    flex-wrap: wrap;
    .tag_item {
      padding: 0 8px;
      margin: 8px 8px 8px 0px;
      flex: 0 1 0;
      font-size: 14px;
    }
    .input_item {
      min-width: 100px;
      flex: 1 1 0;
      border: unset;
      height: auto;
      box-shadow: none;
    }
  }

  .tabs_layout {
    position: absolute;
    width: 100%;
    z-index: 99;
    // float: left;
    background-color: white;
    outline: 0;
    box-shadow: 0 0 4px 0 rgba(117, 117, 117, 0.5);

    // 搜索
    .search_panel {
      overflow: auto;
      height: 30vh;
      width: 100%;
      margin: 0 8px;

      .search_label_item {
        margin: 8px 0;
        padding: 6px;
        font-size: 16px;

        display: inline-block;
      }

      .search_checkbox_panel {
        margin: 0 8px;
        .search_checkbox_item {
          margin: 8px;
          font-size: 16px;
        }
      }
    }

    // 普通
    .tab {
      .ant-tabs-bar {
        margin: 0;
      }
      .tab_panel {
        overflow: auto;
        height: 30vh;
        .lable {
          margin: 10px 0 0px 18px;
          font-size: 18px;
          font-weight: 600;
        }
        .divider {
          margin: 8px 0;
        }

        .checkbox_panel {
          margin: 0 8px;
          .check_item {
            margin: 8px;
            font-size: 16px;
          }

          .label_item {
            margin: 8px 0;
            padding: 6px;
            font-size: 16px;

            display: inline-block;
          }
        }
      }
    }
  }
}

.expand {
  border-color: #40a9ff !important;
  border-right-width: 1px !important;
  outline: 0;
  box-shadow: 0 0 0 2px rgba(24, 144, 255, 0.2);
}

.input_layout:hover {
  border-color: #40a9ff;
  border-right-width: 1px !important;
}

.label_item:hover & {
  border-color: #40a9ff;
  border-right-width: 1px !important;
  outline: 1;
  background: #40a9ff;
  box-shadow: 0 0 0 2px rgba(24, 144, 255, 0.2);
}

.search_label_item:hover & {
  border-color: #40a9ff;
  border-right-width: 1px !important;
  outline: 1;
  background: #40a9ff;
  box-shadow: 0 0 0 2px rgba(24, 144, 255, 0.2);
}
</style>
