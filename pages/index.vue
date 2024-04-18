<template>
  <div class="p-4 pb-12 bg-light-800 dark:bg-dark-5 relative" flex="~ nowrap grow <sm:col">
    <div
      flex="~ col"
      class="relative sticky mr-2 w-24vw min-w-200px <sm:(w-full mr-0 mb-4)"
    >
      <div flex mb-4>
        <!-- 重新计算序号 -->
        <q-btn
          unelevated
          class="ml-2"
          color="blue"
          @click="recalculateOrder"
        >
          <div class="i-fluent:calculator-24-regular mr-2" />
          重新计算序号
        </q-btn>
      </div>

      <textarea
        v-model="outlineText"
        class="bg-white dark:!bg-dark-3 w-full flex-grow p-4 outline-none border-none"
        placeholder="大纲"
        @change="parseOutline"
      />
    </div>
    <draggable
      v-model="outline"
      item-key="id"
      group="chapters"
      class="flex-grow select-none"
      @start="handleDragStart('chapter')"
      @end="handleDragEnd"
    >
      <template #item="{ element }">
        <div
          class="group bg-light-100 dark:bg-dark-1 text-lg mb-2 last:mb-0 overflow-hidden rounded-xl px-4 py-2"
        >
          <div class="flex items-center justify-between text-gray-5">
            <div class="flex items-center">
              <div class="i-fluent:book-24-regular text-lg mr-1" />
              <q-input
                v-model="element.title"
                borderless
                dense
                placeholder="一级标题"
                class="md:w-400px <md:flex-grow"
              />
            </div>
            <div class="hidden group-hover:!flex">
              <q-btn
                flat
                text-color="primary"
                @click.stop="addPage(element)"
                class="mr-1"
              >
                <div class="i-fluent:add-24-regular mr-2" />
                添加二级标题
              </q-btn>
              <q-btn
                unelevated
                color="red"
                @click.stop="removeChapter(element)"
                class=""
              >
                <div class="i-fluent:delete-24-regular mr-2" />
                删除一级标题
              </q-btn>
            </div>
          </div>
          <draggable
            v-model="element.pages"
            item-key="id"
            group="pages"
            @start="handleDragStart('page')"
            @end="handleDragEnd"
            v-auto-animate
          >
            <template #item="{ element: page }">
              <div
                class="group px-4 py-2 bg-light-500 dark:bg-dark-5 text-primary mt-2"
                :class="{ 'opacity-30': isDragging === 'page' }"
              >
                <div class="flex items-center justify-between">
                  <div class="flex items-center">
                    <div class="i-fluent:document-24-regular text-lg mr-3" />
                    <q-input
                      v-model="page.title"
                      outlined
                      dense
                      placeholder="二级标题"
                      class="bg-white dark:!bg-dark-3 bg-opacity-70 md:w-400px"
                    />

                    <!-- 该行要代码、要表格、要流程图、要图表 按钮 -->
                    <div class="flex items-center ml-2">
                      <q-btn
                        flat
                        dense
                        text-color="primary"
                        @click.stop="modifyPointType(page, btn.name)"
                        class="mr-1"
                        v-for="btn in insertBtns.data.value?.data || []"
                        :key="btn.name"
                      >
                        <Icon :name="btn.icon" />
                        <q-tooltip>
                          {{ btn.label }}
                        </q-tooltip>
                      </q-btn>
                    </div>
                  </div>
                  <div class="hidden group-hover:!flex">
                    <q-btn
                      flat
                      dense
                      @click.stop="addPoint(page)"
                      class="text-lg mr-1"
                    >
                      <div class="i-fluent:add-24-regular" />
                    </q-btn>
                    <q-btn
                      flat
                      dense
                      @click.stop="removePage(element, page)"
                      class="text-lg text-red"
                    >
                      <div class="i-fluent:delete-24-regular" />
                    </q-btn>
                  </div>
                </div>
                <q-input
                  borderless
                  v-model="page.pointsText"
                  autogrow
                  type="textarea"
                  :input-style="{ paddingLeft: '1rem' }"
                  class="mt-2"
                  placeholder="在这里输入该节要点，每行一个"
                />
                <draggable
                  v-model="page.points"
                  item-key="id"
                  group="points"
                  @start="handleDragStart('point')"
                  @end="handleDragEnd"
                  v-auto-animate
                >
                  <template #item="{ element: point, index }">
                    <div
                      class="point flex items-center mt-2 ml-2"
                      :class="{ 'opacity-50': isDragging === 'point' }"
                    >
                      <div class="i-mdi:add-circle-outline text-lg mr-2" />
                      <div
                        class="flex-1 transition-all pl-4 rounded-xs bg-transparent duration-300"
                        hover="bg-white dark:!bg-dark-3 ring-2"
                        border="0 solid red"
                        :class="{
                          'border-b-1': !point,
                        }"
                      >
                        <q-input
                          v-model="page.points[index]"
                          borderless
                          dense
                          placeholder="三级标题"
                        />
                      </div>
                      <div class="hidden group-hover:!flex ml-2">
                        <q-btn
                          flat
                          dense
                          @click.stop="removePoint(page, index)"
                          class="text-lg text-red"
                        >
                          <div class="i-fluent:delete-24-regular" />
                        </q-btn>
                      </div>
                    </div>
                  </template>
                </draggable>
              </div>
            </template>
          </draggable>
        </div>
      </template>
    </draggable>
    <q-page-sticky
      position="bottom-right"
      class="z-100"
      :offset="[48, 18]"
      expand
    >
      <div class="q-mt-md scale-120 bg-white dark:!bg-dark-3 rounded-full shadow-lg px-4 py-2">
        <DarkToggle />
        <q-btn flat rounded @click="addChapter" label="添加一级标题">
          <div class="i-fluent:add-24-regular ml-2" />
        </q-btn>
      </div>
    </q-page-sticky>

    <div v-if="outline.length === 0" class="text-center text-lg">
      <div class="i-mdi:book-open-outline text-4xl text-primary mb-2" />
      <div class="text-xl">暂无提纲</div>
      <div>您可以点击“生成提纲”按钮一键生成或在左侧输入框中手动输入</div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { useIntervalFn, useSessionStorage } from '@vueuse/core';
import { ref, watch, nextTick, onMounted, onBeforeUnmount } from 'vue';
import draggable from 'vuedraggable';

type OutlineInsertBtn = {
  name: string;
  icon: string;
  label: string;
}

interface Chapter {
  id?: number;
  title?: string;
  pages: Page[];
}

interface Page {
  id?: number;
  title?: string;
  points: string[];
  pointsText?: string;
}

const isDragging = ref<'chapter' | 'page' | 'point' | null>(null);
const handleDragStart = (type: 'chapter' | 'page' | 'point') => {
  isDragging.value = type;
};
const handleDragEnd = () => {
  isDragging.value = null;
};

const outlineText = ref('')

// 接受 iframe 外部的数据
useEventListener('message', (event) => {
  if (event.data.type === 'outline') {
    outlineText.value = event.data.data;
    parseOutline();
  }
});
// 变更时通知外部
watch(outlineText, () => {
  window.parent.postMessage({ type: 'outline', data: outlineText.value }, '*');
});

const outline = ref<Chapter[]>([]);

const parseOutline = () => {
  const chapters: Chapter[] = [];
  const lines = outlineText.value.split('\n');
  let currentChapter: Chapter | null = null;
  let currentPage: Page | null = null;
  for (const line of lines) {
    if (line.startsWith('# ')) {
      // 新章节
      currentChapter = {
        title: line.slice(2),
        pages: [],
      };
      chapters.push(currentChapter);
      currentPage = null;
    } else if (line.startsWith('## ')) {
      // 新页面
      currentPage = {
        title: line.slice(3),
        points: [],
        pointsText: '',
      };
      if (currentChapter) {
        currentChapter.pages.push(currentPage);
      }
    } else if (line.startsWith('- ')) {
      // 要点
      if (currentPage) {
        currentPage.pointsText += line.slice(2) + '\n';
      }
    } else if (line.startsWith('### ')) {
      // 可能的三级标题
      if (currentPage) {
        currentPage.points.push(line.slice(4));
      }
    }
  }
  outline.value = chapters;
};

const formatOutline = () => {
  let text = '';
  for (const chapter of outline.value) {
    text += `# ${chapter.title}\n`;
    for (const page of chapter.pages) {
      text += `## ${page.title}\n`;
      if (page.pointsText) {
        text += page.pointsText
          .split('\n')
          .map((point) => point.trim())
          .filter((point) => point)
          .map((point) => `- ${point.trim()}`)
          .join('\n');
        text += '\n';
      }
      for (const point of page.points) {
        text += `### ${point}\n`;
      }
    }
  }
  return text;
};

const recalculateOrder = () => {
  let chapterIndex = 1;
  for (const chapter of outline.value) {
    const currentChapterIndex = chapterIndex;

    chapter.title = `第${chapterIndex++}章 ${chapter.title?.replace(
      /第\d+章 /,
      ''
    )}`;

    let pageIndex = 1;
    for (const page of chapter.pages) {
      const currentPageIndex = pageIndex;
      page.title = `${currentChapterIndex}.${pageIndex++} ${page.title?.replace(
        /\d+.\d+ /,
        ''
      )}`;

      let pointIndex = 1;
      for (const point of page.points) {
        page.points[
          pointIndex - 1
        ] = `${currentChapterIndex}.${currentPageIndex}.${pointIndex++} ${point.replace(
          /\d+.\d+.\d+ /,
          ''
        )}`;
      }
    }
  }
};

const saveOutline = useIntervalFn(() => {
  outlineText.value = formatOutline();
}, 5000);

onMounted(() => {
  parseOutline();
  saveOutline.resume();
});

onBeforeUnmount(() => {
  saveOutline.pause();
  outlineText.value = formatOutline();
});

const addChapter = () => {
  outline.value.push({ id: Date.now(), title: '', pages: [] });
};

const removeChapter = (chapter: Chapter) => {
  const index = outline.value.indexOf(chapter);
  outline.value.splice(index, 1);
};

const addPage = (chapter: Chapter) => {
  chapter.pages.push({ id: Date.now(), points: [], pointsText: '' });
};

const removePage = (chapter: Chapter, page: Page) => {
  const index = chapter.pages.indexOf(page);
  chapter.pages.splice(index, 1);
};

const addPoint = (page: Page) => {
  page.points.push('');
};

const removePoint = (page: Page, index: number) => {
  page.points.splice(index, 1);
};

const modifyPointType = (page: Page, type: string) => {
  const index = page.pointsText
    ?.split('\n')
    .findIndex((line) => line.startsWith(type));
  if (index === -1) {
    page.pointsText += (page.pointsText ? '\n' : '') + type;
    page.pointsText = page.pointsText
      ?.split('\n')
      .map((line) => line.trim())
      .filter((line) => line)
      .join('\n');
  } else {
    page.pointsText = page.pointsText
      ?.split('\n')
      .filter((line) => !line.startsWith(type) && line.trim())
      .join('\n');
  }
};

const insertBtns = useFetch<{data: OutlineInsertBtn[]}>('http://sdapi.polars.cc/dialogue/outlineInsertBtn')

</script>
