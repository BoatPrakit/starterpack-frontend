<template>
  <div
    class="flex flex-col justify-center w-full lg:items-center px-10 md:px-20"
  >
    <div class="lg:container">
      <h1 class="flex justify-center mt-5 text-2xl font-extrabold">
        Who are you?
      </h1>
      <CardStyle class="w-full p-5 mt-5 text-left">
        <h2>Total Student: {{ totalUser }}</h2>
      </CardStyle>
      <div class="flex flex-col border items-center mt-3">
        <div class="flex items-center w-full justify-start p-3 py-5">
          <h3 class="text-xl">Add User</h3>
          <button
            class="bg-green-400 p-3 px-5 ml-5 cursor-pointer"
            @click="isShowEdit = true"
          >
            +
          </button>
          <button
            v-if="isShowEdit"
            class="bg-red-400 p-3 px-5 ml-5 cursor-pointer"
            @click="isShowEdit = false"
          >
            Hide
          </button>
        </div>
        <AdminForm v-if="isShowEdit" @adduser="addUser"></AdminForm>
        <p v-if="isShowEdit">Status: {{ status }}</p>
      </div>
      <div
        class="
          flex flex-col
          md:flex-row md:justify-evenly
          flex-wrap
          items-center
          mt-5
        "
      >
        <AdminCard
          v-for="(admin, index) in admins"
          :key="admin._id"
          :name="admin.nickname"
          class="my-3 w-full md:w-1/4"
          :admin-option="getadminOption"
          :index="index"
          @decreaseToTalUser="decreaseTempTotalUser(index)"
        ></AdminCard>
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
      totalUser: 0,
      admins: null,
      tempTotalUser: 0,
      adminOption: {},
      isShowEdit: false,
      status: 'nothing',
    };
  },
  computed: {
    getadminOption() {
      return this.adminOption;
    },
  },
  async created() {
    const resUser = await this.callApi('get', '/api/user');
    const resAdmin = await this.callApi('get', '/api/user/admin');
    this.totalUser = resUser.data.length;
    this.admins = resAdmin.data;
    this.tempTotalUser = this.totalUser;
  },
  methods: {
    decreaseTempTotalUser(index) {
      const decreaseEach = Math.ceil(this.totalUser / this.admins.length);
      const amountUser =
        this.tempTotalUser - decreaseEach >= 0
          ? decreaseEach
          : this.tempTotalUser;
      const start = index === 0 ? 1 : this.adminOption[index - 1].end + 1;
      const end = start + (decreaseEach - 1);
      this.adminOption[index] = {
        amountUser,
        start,
        end,
      };
      this.tempTotalUser -= decreaseEach;
    },
    async addUser(payload) {
      if (payload) {
        const res = await this.callApi('post', '/api/user', payload);
        if (res) {
          this.status = res.status;
          if (res.status === 201) {
            this.totalUser += 1;
          }
        }
        setTimeout(() => {
          this.status = 'nothing';
        }, 2000);
      }
    },
  },
};
</script>
