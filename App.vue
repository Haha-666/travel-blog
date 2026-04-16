<template>
  <div class="min-h-screen bg-slate-50 p-4 md:p-8 font-sans text-slate-900">
    
    <div v-if="currentMode === 'list'" class="max-w-5xl mx-auto bg-white shadow-xl rounded-2xl overflow-hidden border border-slate-200">
      <header class="bg-slate-900 p-6 text-white flex justify-between items-center">
        <h1 class="text-2xl font-bold tracking-tight">旅遊部落格</h1>
        <button @click="goToForm()" class="bg-blue-600 hover:bg-blue-700 text-white font-semibold px-6 py-2 rounded-lg shadow-sm transition transform active:scale-95">
          + 新增貼文
        </button>
      </header>

      <div class="p-6">
        <div class="mb-6">
          <div class="relative w-full md:w-96">
            <span class="absolute inset-y-0 left-0 pl-3 flex items-center text-slate-400">🔍</span>
            <input v-model="searchQuery" type="text" placeholder="搜尋標題或內容..." class="w-full border border-slate-200 rounded-lg pl-10 pr-4 py-2 focus:ring-2 focus:ring-blue-500 outline-none" @input="currentPage = 1" />
          </div>
        </div>

        <div class="overflow-x-auto border border-slate-100 rounded-lg">
          <table class="w-full text-left">
            <thead>
              <tr class="bg-slate-50 border-b border-slate-100 text-slate-500 text-sm uppercase">
                <th class="p-4 font-semibold">編號</th>
                <th class="p-4 font-semibold">文章標題 (點擊閱讀)</th>
                <th class="p-4 font-semibold">操作</th>
              </tr>
            </thead>
            <tbody class="divide-y divide-slate-100">
              <tr v-for="(post, index) in paginatedPosts" :key="post.id" 
                  @click="goToDetail(post)"
                  class="hover:bg-blue-50/50 transition cursor-pointer">
                <td class="p-4 text-slate-400 font-mono text-sm">
                  #{{ filteredPosts.length - ((currentPage - 1) * pageSize + index)}}
                </td>
                <td class="p-4 font-bold text-blue-600 hover:text-blue-800 underline underline-offset-4">{{ post.title }}</td>
                <td class="p-4" @click.stop> <div class="flex gap-3">
                    <button @click="goToForm(post)" class="text-slate-500 hover:text-blue-600 font-medium">編輯</button>
                    <button @click="deletePost(post.id)" class="text-red-400 hover:text-red-700 font-medium">刪除</button>
                  </div>
                </td>
              </tr>
              <tr v-if="paginatedPosts.length === 0">
                <td colspan="3" class="p-12 text-center text-slate-400 italic">目前沒有找到任何文章...</td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="flex flex-col md:flex-row justify-between items-center mt-6 gap-4">
          <p class="text-slate-500 text-sm">共 {{ filteredPosts.length }} 筆資料</p>
          <nav class="flex items-center gap-1">
            <button @click="currentPage--" :disabled="currentPage === 1" class="px-4 py-1.5 border rounded-md disabled:opacity-30">上一頁</button>
            <button v-for="p in totalPages" :key="p" @click="currentPage = p" :class="['w-10 h-10 rounded-md', currentPage === p ? 'bg-blue-600 text-white shadow-md' : 'hover:bg-slate-100 text-slate-600']">{{ p }}</button>
            <button @click="currentPage++" :disabled="currentPage === totalPages" class="px-4 py-1.5 border rounded-md disabled:opacity-30">下一頁</button>
          </nav>
        </div>
      </div>
    </div>

    <div v-else-if="currentMode === 'detail'" class="max-w-4xl mx-auto">
      <button @click="currentMode = 'list'" class="mb-4 text-blue-600 hover:text-blue-800 font-semibold flex items-center gap-2 transition">
        ← 返回文章列表
      </button>
      
      <article class="bg-white shadow-2xl rounded-2xl overflow-hidden border border-slate-200">
        <header class="bg-slate-900 p-8 text-white">
          <h1 class="text-3xl font-bold mb-4">{{ currentPost.title }}</h1>
          <div class="flex items-center text-slate-400 text-sm font-mono">
            <span>文章編號：#{{ currentPost.id }}</span>
          </div>
        </header>
        
        <div class="p-8 md:p-12">
          <p class="text-slate-700 leading-relaxed text-lg whitespace-pre-wrap">{{ currentPost.body }}</p>
          
          <div class="mt-12 pt-6 border-t border-slate-100 flex justify-end gap-4">
            <button @click="goToForm(currentPost)" class="px-6 py-2 border border-slate-200 rounded-lg hover:bg-slate-50 transition">重新編輯</button>
            <button @click="currentMode = 'list'" class="px-6 py-2 bg-slate-900 text-white rounded-lg hover:bg-slate-800 transition shadow-lg">看完了，回首頁</button>
          </div>
        </div>
      </article>
    </div>

    <div v-else-if="currentMode === 'form'" class="max-w-3xl mx-auto bg-white shadow-2xl rounded-2xl overflow-hidden border border-slate-200">
      <div class="p-6 border-b border-slate-100 bg-slate-900 text-white flex justify-between items-center">
        <h2 class="text-xl font-bold">{{ isEditing ? '📝 編輯文章' : '✨ 撰寫新文章' }}</h2>
        <button @click="currentMode = 'list'" class="text-slate-400 hover:text-white font-bold text-xl">&times;</button>
      </div>
      <div class="p-8 space-y-6">
        <div>
          <label class="block text-sm font-bold text-slate-700 mb-2">文章標題</label>
          <input v-model="currentPost.title" type="text" class="w-full border border-slate-200 rounded-xl px-4 py-3 focus:ring-2 focus:ring-blue-500 outline-none shadow-sm" />
        </div>
        <div>
          <label class="block text-sm font-bold text-slate-700 mb-2">文章內容</label>
          <textarea v-model="currentPost.body" rows="12" class="w-full border border-slate-200 rounded-xl px-4 py-3 focus:ring-2 focus:ring-blue-500 outline-none shadow-sm"></textarea>
        </div>
        <div class="flex justify-end gap-4 pt-4">
          <button @click="currentMode = 'list'" class="px-6 py-2 text-slate-500 font-semibold">取消返回</button>
          <button @click="savePost" class="px-10 py-2 bg-blue-600 hover:bg-blue-700 text-white font-bold rounded-xl shadow-lg transition transform active:scale-95">發佈文章</button>
        </div>
      </div>
    </div>

  </div>
</template>

<script setup>
import { ref, computed, onMounted, watch } from 'vue';

const DB_KEY = 'VUE_APP_POSTS';
const posts = ref([
  { id: 1, title: "日本東京", body: "東京神社分享" },
  { id: 2, title: "韓國釜山", body: "韓國釜山5日遊" },
  { id: 3, title: "美國加州", body: "加州美食、住宿、景點"},
  { id: 4, title: "日本沖繩", body: "沖繩5天4夜行程" },
  { id: 5, title: "日本北海道", body: "日本北海道景點分享" }
]);

const currentMode = ref('list'); // 'list', 'form', 'detail'
const searchQuery = ref('');
const currentPage = ref(1);
const pageSize = 5;
const isEditing = ref(false);
const currentPost = ref({ id: null, title: '', body: '' });

// --- 功能方法 ---
const goToDetail = (post) => {
  currentPost.value = { ...post };
  currentMode.value = 'detail';
};

const goToForm = (post = null) => {
  if (post) {
    isEditing.value = true;
    currentPost.value = { ...post };
  } else {
    isEditing.value = false;
    currentPost.value = { id: null, title: '', body: '' };
  }
  currentMode.value = 'form';
};

const savePost = () => {
  if (!currentPost.value.title.trim() || !currentPost.value.body.trim()) {
    alert('請填寫完整內容');
    return;
  }
  if (isEditing.value) {
    const idx = posts.value.findIndex(p => p.id === currentPost.value.id);
    posts.value[idx] = { ...currentPost.value };
  } else {
    const newId = posts.value.length > 0 ? Math.max(...posts.value.map(p => p.id)) + 1 : 1;
    posts.value.unshift({ ...currentPost.value, id: newId });
    currentPage.value = 1;
  }
  currentMode.value = 'list';
};

const deletePost = (id) => {
  if (confirm('確定要刪除這篇文章嗎？')) {
    posts.value = posts.value.filter(p => p.id !== id);
    if (paginatedPosts.value.length === 0 && currentPage.value > 1) {
      currentPage.value--;
    }
  }
};

// --- 資料持久化與計算屬性 (維持原樣) ---
const loadData = () => {
  const saved = localStorage.getItem(DB_KEY);
  if (saved) posts.value = JSON.parse(saved);
};
watch(posts, (newVal) => localStorage.setItem(DB_KEY, JSON.stringify(newVal)), { deep: true });
const filteredPosts = computed(() => {
  let result = [...posts.value];
  if (searchQuery.value) {
    const query = searchQuery.value.toLowerCase();
    result = result.filter(p => p.title.toLowerCase().includes(query) || p.body.toLowerCase().includes(query));
  }
  return result.sort((a, b) => b.id - a.id);
});
const totalPages = computed(() => Math.ceil(filteredPosts.value.length / pageSize) || 1);
const paginatedPosts = computed(() => {
  const start = (currentPage.value - 1) * pageSize;
  return filteredPosts.value.slice(start, start + pageSize);
});
onMounted(loadData);
</script>