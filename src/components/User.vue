import {Gender} from "../model/User";
<template>
  <div class="user-container">

    <div class="users">
      <h3>Users: </h3>
      <div v-for="user of users" :key="user.id">
        <span>{{user.name}} - {{user.gender}}</span>
        <i class="icon el-icon-delete" @click="deleteUser(user.id)"></i>
      </div>
    </div>

    <hr />

    <div class="user-create">
      <h3>User create: </h3>
      <div class="row">
        <label for="name">Name: </label>
        <el-input id="name" type="text" v-model="name" />
      </div>

      <div class="row">
        <label>Gender: </label>
        <el-radio v-model="gender" :label="Gender.MALE">Male</el-radio>
        <el-radio v-model="gender" :label="Gender.FEMALE">Female</el-radio>
      </div>

      <div class="row">
        <el-button class="create-btn" type="default" @click="createUser">Submit</el-button>
      </div>
    </div>
  </div>
</template>

<script lang="ts">
import {Component, Prop, Vue} from "vue-property-decorator";
import UserService from "../services/UserService";
import {Convoy} from "../utils/axios";
import {Gender} from "../model/User";

const userService = new UserService();

@Component
export default class HelloWorld extends Vue {
  @Prop() private msg!: string;

  public Gender = Gender;
  public users = [];
  public name = '';
  public gender = Gender.MALE;

  public created() {
    this.fetchUsers();

    setTimeout(() => {
      this.fetchUsers();
    }, 1000)
  }

  @Convoy
  public fetchUsers() {
    return userService.fetchUsers()
      .then((resp) => {
        this.users = resp.data;
      });
  }

  public deleteUser(id: number) {
    return userService.deleteUser(id, Gender.MALE)
      .then((resp) => {
        console.log(resp);
        console.log('I want to handle the error by myself.');
      });
  }

  // PROMISE CALL
  // @Convoy
  // public createUser() {
  //   return userService.createUser({
  //     name: this.name,
  //     gender: this.gender
  //   });
  // }

  // ASYNC CALL
  @Convoy
  public async createUser() {
    try {
      const data = await userService.createUser({
        name: this.name,
        gender: this.gender
      });
      console.log(data)
    } catch (error) {
      console.log('I just want to catch the 403 error, throw the others out!');

      if (error.response && error.response.status === '403') {
        console.error('Forbidden');
        return;
      }

      throw error;
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
.user-container {
  text-align: left;
}

.icon {
  margin-left: 20px;
  cursor: pointer;
}

hr {
  margin: 20px 0;
}

.user-create {
  width: 400px;

  .row {
    display: flex;
    align-items: center;
    margin-bottom: 10px;

    label {
      flex: 1;
    }

    .el-input {
      flex: 4;
    }
  }

  .create-btn {
    width: 100%;
  }
}
</style>
