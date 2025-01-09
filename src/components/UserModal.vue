<template>
  <div class="modal-overlay">
    <div class="modal-content">
      <h2>{{ isEdit ? "Edit User" : "Create User" }}</h2>
      <form @submit.prevent="handleSave">
        <div class="form-inputs">
          <label class="bloc-form">
            Username:
            <input type="text" v-model="localUser.username" :disabled="isEdit" required />
          </label>
          <label class="bloc-form">
            Roles:
            <input
                type="text"
                v-model="rolesInput"
                placeholder="Comma-separated roles"
                required
            />
          </label>
          <label class="bloc-form">
            Timezone:
            <input type="text" v-model="localUser.preferences.timezone" required />
          </label>
          <label>
            Is Active?
            <input type="checkbox" v-model="localUser.active" />
          </label>
        </div>
        <div class="form-button">
          <button type="submit">{{ isEdit ? "Save Changes" : "Create User" }}</button>
          <button type="button" @click="$emit('close')">Cancel</button>
        </div>
      </form>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    user: {
      type: Object,
      default: () => ({ preferences: {} }),
    },
    isEdit: {
      type: Boolean,
      default: false,
    },
  },
  data() {
    return {
      localUser: { ...this.user, preferences: { ...this.user.preferences } },
      rolesInput: this.user.roles ? this.user.roles.join(", ") : "",
    };
  },
  methods: {
    handleSave() {
      const updatedUser = {
        ...this.localUser,
        roles: this.rolesInput.split(",").map((role) => role.trim()),
      };
      this.$emit("save", updatedUser);
    },
  },
  watch: {
    user: {
      immediate: true,
      handler(newUser) {
        this.localUser = { ...newUser, preferences: { ...newUser.preferences } };
        this.rolesInput = newUser.roles ? newUser.roles.join(", ") : "";
      },
    },
  },
};
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
}

.modal-content {
  background: #fff;
  padding: 20px;
  border-radius: 5px;
  width: 400px;
}
form {
  display: flex;
  flex-direction: column;
  flex-wrap: wrap;
  justify-content: space-around;
  align-items: center;
}
.form-inputs {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  align-content: center;
  justify-content: space-between;
  align-items: center;
}
.bloc-form {
  display: flex;
  flex-direction: column;
  align-items: flex-start;
  margin-bottom: 15px;
}
input {
  border: none;
  padding: 10px;
  border-bottom: 1px solid #ddd;
}
.form-button {
  display: flex;

}
button{
  margin: 0 5px;
  padding: 10px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}
button:first-child{
  background: #42b983;
  color: white;
  font-weight: 700;
}
button:first-child:hover{
  opacity: 0.4;
  color: black;
}
button:last-child{
  background: orangered;
  color: white;
  font-weight: 700;
}
button:last-child:hover{
  opacity: 0.4;
  color: black;
}
</style>
