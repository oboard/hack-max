<template>
  <div class="flex flex-col gap-2 items-center justify-start">
    <p>按回车键清空
      111{{ selectedDataset }}111</p>

    <div class="join w-full">
      <select class="select select-bordered join-item" @change="(e) => this.dataset = this.datasetList[e.target.value]">
        <option v-for="item in Object.keys(datasetList)" :selected="item == selectedDataset" :value="item">{{ item }}
        </option>
      </select>
      <input type="text" className="input input-bordered join-item w-full" v-model="inputStr"
        @keyup="() => getFirstLetter()" @keyup.enter="() => inputStr = ''" placeholder="请输入一串首字母">
      <input v-for="(item, index) in radioData" :key="index" class="join-item btn" type="radio" name="options"
        :aria-label="item.value" @change="getRadioVal(index)" :checked="radioIndex == index" />
    </div>
    <div class="card bg-base-100 shadow-md w-full" v-for="result in results">
      <div class="card-body">
        <p class="text-red-500 font-bold text-2xl md:text-6xl">{{ result.answer }}</p>
        <p>{{ result.topic }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import { pinyin } from "pinyin";

import { xinli } from './datasets/xinli.js'
import { max } from './datasets/max.js'

export default {
  data() {
    return {
      inputStr: '',
      selectedDataset: '马克思',
      datasetList: { '马克思': max, '心理': xinli, '军事理论': junshi },
      results: [

      ],
      radioData: [
        { value: '全部' },
        // { value: '单选' },
        // { value: '多选' }
      ],
      radioVal: '全部',// 用于设置默认选中项
      dataset: max,
      radioIndex: 0,
    }
  },
  methods: {
    generate(raw) {
      let list = raw.split(/\d+(、|\.)/g);
      let firstLetter = "";
      let count = 0;

      for (let index = 0; index < list.length; index++) {
        const i = list[index];

        console.log(count);
        count++;
        // if (count > 266) continue;
        var items = i.split(/答案：/g);
        let str = pinyin(
          items[0]
            .split("A")[0]
            .trim()
            .slice(0, i.length > 30 ? 20 : i.length),
          {
            style: "first_letter",
            compact: true,
            heteronym: true,
          }
        );
        firstLetter = "";
        for (let i = 0; i < str.length; i++) {
          for (let j = 0; j < str[i].length; j++) {
            firstLetter += str[i][j];
          }
          firstLetter += "/";
        }
        let ans = items[0].split(/A|B|C|D|E|F|G|H/g);
        console.log(items[1]);
        let composeAnswer = "";
        if (items.length > 1) {
          const reals = items[1].replace(/\n/g, "").trim().split("");
          if (reals.length == ans.length - 1 && reals.length != 1) {
            composeAnswer = "全选";
          } else {
            for (let index = 1; index < ans.length; index++) {
              const e = ans[index];
              const pre = ["A", "B", "C", "D", "E", "F", "G"][index - 1];
              for (let realIndex = 0; realIndex < reals.length; realIndex++) {
                const real = reals[realIndex];
                if (pre == real) {
                  composeAnswer += pre + e;
                }
              }
            }
          }
          dataset.push({
            topic: items[0],
            answer: composeAnswer,
            pinyin: firstLetter,
          });
        }
      }
    },
    getRadioVal(index) {
      this.radioIndex = index;
      console.log(this.radioVal);
      getFirstLetter();
    },
    getFirstLetter() {
      let start = 0, end = this.dataset.length;
      this.results = [];
      if (this.inputStr.trim() == '') return;
      if (this.radioIndex == 1) {
        end = 238;
      } else if (this.radioIndex == 2) {
        start = 238;
      }
      for (let index = start; index < end; index++) {
        const item = this.dataset[index];
        console.log(item);
        if (item['pinyin'].indexOf(this.inputStr.trim()) >= 0) {
          this.results.push(item);
          // return;
        }
      }
    },
  }
}
</script>