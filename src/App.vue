<template>
  <a-layout style="min-height: 100vh">
    <a-layout-content>
      <a-collapse v-model:activeKey="activeKey" accordion>
        <a-collapse-panel key="1" header="印">
          <a-checkbox-group
            v-model:value="selectType"
            name="checkboxgroup"
            :options="types"
          />
        </a-collapse-panel>
        <a-collapse-panel key="2" header="レア">
          <a-checkbox-group
            v-model:value="selectRare"
            name="checkboxgroup"
            :options="rares"
          />
        </a-collapse-panel>
      </a-collapse>

      <a-table
        :columns="columns"
        :data-source="computedChars"
        :pagination="{
          defaultPageSize: 30,
        }"
      >
        <template #type="{ text: type }">
          <span>
            <a-tag v-for="t in type" :key="t" color="green">
              {{ t }}
            </a-tag>
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
</template>

<script>
import { defineComponent, ref } from "vue";
import TypeData from "./data/type.js";
import CharData from "./data/characters.js";

export default defineComponent({
  name: "App",
  components: {},
  setup() {
    // レア度
    const rares = Object.keys(CharData).map((rare) => ({
      label: rare,
      value: rare,
    }));

    // 印
    const types = TypeData.map((type) => {
      return {
        label: type.name,
        value: type.name,
      };
    });

    const chars = ref([]);

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

    return {
      types,
      rares,
      chars,
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
