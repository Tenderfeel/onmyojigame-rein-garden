<template>
  <a-layout style="min-height: 100vh" :hasSider="true">

  <!-- content -->
  <a-layout>
    <a-layout-content>
      <!-- Table -->
      <a-table
        :columns="columns"
        :data-source="computedChars"
        size="small"
        :pagination="{
          defaultPageSize: 30,
        }"
      >
        <template #type="{ text: type }">
          <span>
            <TypeIcon v-for="t in type" :key="t" :data="t" />
          </span>
        </template>
        <template #action="{ record }">
          <a-switch v-model:checked="record.completed" @click="handleSwitch" />
        </template>
      </a-table>
    </a-layout-content>
    <a-layout-footer style="padding: 1rem">
      <div style="text-align: center">
        <a href="https://twitter.com/vesperfeel">&copy; Vesperfeel</a>
      </div>
    </a-layout-footer>
  </a-layout>

    <a-layout-sider 
      breakpoint="md"
      collapsed-width="0"
      :zeroWidthTriggerStyle="{left: '-36px', 'border-radius': '2px 0 0 2px'}"
      :style="{ height: '100vh', position: 'sticky', right: 0, top:0, zIndex: 1000 }"
      :width="siderWidth"
      @breakpoint="onBreakpoint"
    >
    <a-card :style="{ margin: '8px'}" size="small">
      <a-statistic title="再覚醒" :value="completed" :value-style="{ fontSize: '20px' }">
        <template #suffix>
          <span style="font-size: 14px">/ {{chars.length}}</span>
        </template>
      </a-statistic>
    </a-card>

    <a-card title="印" :style="{ margin: '8px'}" size="small">
      <a-checkbox-group
        v-model:value="selectType"
        name="checkboxgroup"
      >
      <a-row :gutter="[8, { xs: 8, sm: 16, md: 16, lg: 16 }]">
        <a-col v-for="type in types" :key="type.id" :xs="24" :sm="12">
          <a-checkbox :value="type.name">
            <TypeIcon :data="type.name" />
          </a-checkbox>
        </a-col>
      </a-row>
      </a-checkbox-group>
    </a-card>

    <a-card title="レア" :style="{ margin: '8px'}" size="small">
      <a-checkbox-group
          v-model:value="selectRare"
          name="checkboxgroup"
      >
      <a-row :gutter="[8, { xs: 8, sm: 16, md: 16, lg: 16 }]">
        <a-col v-for="rare in rares" :key="rare" :xs="24" :sm="12">
          <a-checkbox :value="rare">
            {{rare}}
          </a-checkbox>
        </a-col>
      </a-row>

       </a-checkbox-group>
    </a-card>
    </a-layout-sider>

  </a-layout>
</template>

<script>
import { defineComponent, ref } from "vue";
import TypeData from "./data/type.js";
import CharData from "./data/characters.js";

import TypeIcon from "./components/TypeIcon.vue";

export default defineComponent({
  name: "App",
  components: { TypeIcon },
  setup() {
    // レア度
    const rares = Object.keys(CharData);

    // 印
    const types = [...TypeData];

    const chars = ref([]);

    const siderWidth = ref(200)

  // 式神初期データ
    const defaultData = Object.entries(CharData).reduce(
      (accu, [rare, list]) => {
        return accu.concat(
          list.map(({ name, type }) => {
            return { key: name, rare, name, type, completed: false };
          })
        );
      },
      []
    );

    const logData = JSON.parse(
      localStorage.getItem("rein-garden.chars") || "[]"
    );

    if (logData.length) {
      chars.value = logData;
    } else {
      chars.value = defaultData;
    }

    const onBreakpoint = broken => {
      console.log(broken);
      siderWidth.value = broken ? 110 : 200
    };

    return {
      types,
      rares,
      chars,
      onBreakpoint,
      siderWidth
    };
  },
  data() {
    return {
      activeKey: [],
      selectType: [],
      selectRare: Object.keys(CharData),
      columns: [
        {
          title: "レア",
          dataIndex: "rare",
          key: "rare",
        },
        {
          title: "式神",
          dataIndex: "name",
          key: "name",
        },
        {
          title: "印",
          dataIndex: "type",
          key: "type",
          slots: { customRender: "type" },
        },
        {
          title: "再覚醒",
          key: "action",
          slots: { customRender: "action" },
          filters: [
            {
              text: '覚醒済み',
              value: 1,
            },
            {
              text: '未覚醒',
              value: 0,
            },
          ],
          onFilter: (value, record) => record.completed === Boolean(value),
        },
      ],
    };
  },
  computed: {
    // chars
    computedChars() {
      const chars = this.chars.filter(({ type, rare }) => {
        const typeCheck = this.selectType.every((st) => {
          return type.includes(st);
        });
        const rareCheck = this.selectRare.includes(rare);
        return typeCheck && rareCheck;
      });

      return chars;
    },

    // 再覚醒達成率
    completed() {
      return this.chars.reduce((acc, cur) => {
        return acc + cur.completed
      }, 0)
    },

  },
  methods: {
    handleSwitch() {
      localStorage.setItem("rein-garden.chars", JSON.stringify(this.chars));
    },
  },
});
</script>

<style>
</style>
