<template>
  <div v-if="!isLoading" class="p-5 flex justify-center">
    <div class="md:w-1/2 lg:px-10 flex flex-col">
      <div class="flex flex-col w-full items-center">
        <h1 class="text-2xl font-bold">Admin</h1>
        <h2 class="text-xl font-semibold my-3">{{ name }}</h2>
      </div>
      <h1 v-if="!userInResponsibilty.length">No user for give feedback</h1>
      <div class="flex flex-col flex-wrap w-full">
        <AdminFeedbackForm
          v-for="user in userInResponsibilty"
          :key="user._id"
          :std-id="user.std_id"
          :feedbacks="user.feedbacks"
          :user-id="user._id"
          @sendfeedback="postFeedbackToServer"
          @updatefeedback="putFeedbackToServer"
          @deletefeedback="deleteFeedbackToServer"
        ></AdminFeedbackForm>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  middleware({ store, redirect }) {
    const user = store.$auth.user;
    if (user.role === 'user') {
      redirect('/feedback');
    } else if (!user) redirect('/');
  },
  data() {
    return {
      start: this.$route.query.start,
      name: this.$route.params.name,
      end: this.$route.query.end,
      userInResponsibilty: [],
      isLoading: true,
    };
  },
  created() {
    this.fetchData();
    this.isLoading = false;
  },
  methods: {
    async postFeedbackToServer({ userId, comment, day, commentBy }) {
      const res = await this.callApi('post', `/api/user/${userId}/feedback`, {
        comment,
        commentBy,
        day,
      });
      if (res) {
        if (res.status >= 200) {
          this.fetchData();
        }
      }
    },
    async fetchData() {
      const res = await this.callApi(
        'get',
        `/api/user?start=${this.start}&end=${this.end}`
      );
      if (res) {
        if (res.status >= 200) {
          const limitUser = res.data.docs;
          this.userInResponsibilty = limitUser.sort(
            (a, b) => Number(a.std_id) - Number(b.std_id)
          );
        }
      }
    },
    async putFeedbackToServer({ comment, day, commentBy, feedbackId }) {
      if (feedbackId) {
        const payload = { comment, day, commentBy };
        const res = await this.callApi(
          'put',
          `/api/feedback/${feedbackId}`,
          payload
        );
        if (res.status >= 200) {
          this.fetchData();
        }
      }
    },
    async deleteFeedbackToServer({ feedbackId, userId }) {
      const res = await this.callApi(
        'delete',
        `/api/user/${userId}/feedback/${feedbackId}`
      );
      if (res.status >= 200) {
        this.fetchData();
      }
    },
  },
};
</script>

<style></style>
