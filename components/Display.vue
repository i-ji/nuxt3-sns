<template>
  <v-card>
    <v-tabs v-model="tab" align-tabs="center" fixed-tabs>
      <v-tab :value="recommendation">おすすめ</v-tab>
      <v-tab :value="follow">フォロー中</v-tab>
    </v-tabs>

    <v-card-text>
      <v-window v-model="tab">
        <v-window-item :value="recommendation">
          <v-container>
            <v-card
              v-for="post in randomPosts"
              :key="post.id"
              class="mb-10"
              :title="post.name"
              :subtitle="post.title"
            >
              <template v-slot:prepend>
                <v-icon icon="mdi-account" color="primary"></v-icon>
              </template>

              <v-card-text>{{ post.body }}</v-card-text>

              <v-card-actions>
                <v-btn
                  class="bg-black text-white font-bold"
                  rounded="xl"
                  @click="toFollow(post.userId)"
                  v-show="!post.isFollow"
                >
                  フォローをする
                </v-btn>
                <v-btn
                  class="font-bold"
                  variant="outlined"
                  rounded="xl"
                  @click="toFollow(post.userId)"
                  v-show="post.isFollow"
                >
                  フォローを外す
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-container>
        </v-window-item>

        <v-window-item value="follow">
          <v-container class="min-h-96">
            <v-card
              v-for="follow in postFollows"
              :key="follow.id"
              class="mb-10"
              :title="follow.name"
              :subtitle="follow.title"
            >
              <template v-slot:prepend>
                <v-icon icon="mdi-account" color="primary"></v-icon>
              </template>

              <v-card-text>{{ follow.body }}</v-card-text>

              <v-card-actions>
                <v-btn
                  variant="outlined"
                  class="font-bold"
                  rounded="xl"
                  @click="deleteFollow(follow.userId)"
                >
                  フォローを外す
                </v-btn>
              </v-card-actions>
            </v-card>
          </v-container>
        </v-window-item>
      </v-window>
    </v-card-text>
  </v-card>
</template>

<script setup lang="ts">
// API
interface Post {
  userId: number;
  id: number;
  title: string;
  body: string;
  isFollow?: boolean;
  name?: string;
}
const { data: posts } = await useFetch<Post[]>(
  "https://jsonplaceholder.typicode.com/posts"
)!;

// APIに独自のオブジェクトを追加
if (posts.value !== null) {
  for (let i = 0; i < posts.value.length; i++) {
    posts.value[i].isFollow = false;
    posts.value[i].name = `User${posts.value[i].userId}`;
  }
}

// postsの並び替え
let randomPosts = ref<Post[]>([]);
const shuffleArray = (array: Post[]): Post[] => {
  randomPosts.value = [...array];

  for (let i = randomPosts.value.length - 1; i >= 0; i--) {
    let rand = Math.floor(Math.random() * (i + 1));
    // 配列の要素の順番を入れ替える
    let tmpStorage = randomPosts.value[i];
    randomPosts.value[i] = randomPosts.value[rand];
    randomPosts.value[rand] = tmpStorage;
  }
  return randomPosts.value;
};

if (posts.value !== null) {
  shuffleArray(posts.value);
}

// タブメニュー
const tab = ref<null>(null);
const recommendation = ref<string>("recommendation");
const follow = ref<string>("follow");

// フォロー
let follows = ref<Post[] | undefined>([]);
let postFollows = ref<Post[] | undefined>([]);
let removeFollows = ref<Post[] | undefined>([]);

const toFollow = (userId: number) => {
  follows.value = posts.value?.filter((post) => post.userId === userId);
  follows.value?.forEach((follow) => {
    postFollows.value?.unshift(follow);
    follow.isFollow = !follow.isFollow;
    if (follow.isFollow === false && postFollows.value !== undefined) {
      for (let i = postFollows.value.length - 1; i >= 0; i--) {
        if (postFollows.value[i].userId === userId) {
          postFollows.value.splice(i, 1);
          // removeFollows.value?.unshift(postFollows.value.splice(i, 1)[0]);
        }
      }
    }
  });
};

// フォローを外す (仮)
const deleteFollow = (userId: number) => {
  if (postFollows.value !== undefined) {
    for (let i = postFollows.value.length - 1; i >= 0; i--) {
      if (postFollows.value[i].userId === userId) {
        // postFollows.value.splice(i, 1);
        removeFollows.value?.unshift(postFollows.value.splice(i, 1)[0]);
        removeFollows.value?.forEach((post) => {
          post.isFollow = false;
        });
        removeFollows.value = [];
        // console.log(removeFollows.value);
      }
    }
  }
};
</script>
