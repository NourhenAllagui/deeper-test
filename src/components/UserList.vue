<template>
  <h1>Deeper Systems Users List</h1>
  <div class="container">
    <div class="add-botton">
      <button @click="openCreateModal" class="btn btn-success">Create User</button>
    </div>
   <div class="content">
    <table class="styled-table">
      <thead>
      <tr>
        <th>Username</th>
        <th>Roles</th>
        <th>Timezone</th>
        <th>Is Active?</th>
        <th>Last Updated At</th>
        <th>Created At</th>
        <th>Actions</th>
      </tr>
      </thead>
      <tbody>
      <tr v-for="user in paginatedUsers" :key="user.username">
        <td>
          <a :href="'/user/' + user.username" class="username-link">{{ user.username }}</a>
        </td>
        <td>{{ user.roles ? user.roles.join(', ') : '' }}</td>
        <td>{{ user.preferences.timezone }}</td>
        <td>{{ user.active ? 'Yes' : 'No' }}</td>
        <td>{{ user.lastUpdated ? new Date(user.lastUpdated).toLocaleString() : new Date(user.created_ts * 1000).toLocaleString() }}</td>
        <td>{{ new Date(user.created_ts * 1000).toLocaleString() }}</td>
        <td class="table-button">
          <button @click="openEditModal(user)">Edit</button>
          <button @click="confirmDelete(user)">Delete</button>
        </td>
      </tr>
      </tbody>
    </table>
  <div class="pagination">
      <button @click="prevPage" :disabled="currentPage === 1">Previous</button>
      <span>Page {{ currentPage }} of {{ totalPages }}</span>
      <button @click="nextPage" :disabled="currentPage === totalPages">Next</button>
    </div>
    <user-modal
        v-if="modalVisible"
        :user="modalUser"
        :is-edit="isEdit"
        @close="closeModal"
        @save="saveUser"
    />
  </div>
  </div>
</template>

<script>
import axios from 'axios';
import UserModal from "@/components/UserModal";

export default {
  data() {
    return {
      users: [],
      currentPage: 1,
      itemsPerPage: 10,
      modalVisible: false,
      isEdit: false,
      modalUser: {},
    };
  },
 components: { UserModal },
  computed: {
    totalPages() {
      return Math.ceil(this.users.length / this.itemsPerPage);
    },
    paginatedUsers() {
      const start = (this.currentPage - 1) * this.itemsPerPage;
      const end = start + this.itemsPerPage;
      return this.users.slice(start, end);
    },
  },
  methods: {
    fetchUsers() {
      axios.get('http://localhost:5000/users')
          .then(response => {
            this.users = response.data;
          })
          .catch(error => {
            console.error('Error fetching users:', error);
          });
    },
    nextPage() {
      if (this.currentPage < this.totalPages) {
        this.currentPage++;
      }
    },
    prevPage() {
      if (this.currentPage > 1) {
        this.currentPage--;
      }
    },
    openCreateModal() {
      this.isEdit = false;
      this.modalUser = { username: "", roles: [], preferences: { timezone: "" }, active: true };
      this.modalVisible = true;
    },
    openEditModal(user) {
      this.isEdit = true;
      this.modalUser = { ...user };
      this.modalVisible = true;
    },
    confirmDelete(user) {
      if (confirm(`Are you sure you want to delete ${user.username}?`)) {
        axios.delete(`http://localhost:5000/users/${user._id}`)
            .then(() => {
              this.fetchUsers();
            })
            .catch(error => {
              console.error('Error deleting user:', error);
            });
      }
    },
    closeModal() {
      this.modalVisible = false;
      this.modalUser = {};
    },
    saveUser(user) {
      const timestamp = new Date().toLocaleString();
      if (this.isEdit) {
        axios.put(`http://localhost:5000/users/${user._id}`, {
          username: user.username,
          roles: user.roles,
          active: user.active,
          preferences: {
            timezone:user.preferences.timezone,
          }, lastUpdated: timestamp
        })
            .then(response => {
              console.log('User updated successfully:', response.data);
              const updatedUser = response.data;
              const index = this.users.findIndex(u => u._id === updatedUser._id);
              if (index !== -1) {
                this.users[index] = {updatedUser, lastUpdated: timestamp};
              }
              this.fetchUsers();
              this.closeModal();
            })
            .catch(error => {
              console.error('Error updating user:', error);
            });
      } else {
        axios.post('http://localhost:5000/users', {
        ...user,
        created_ts: Math.floor(Date.now() / 1000),
      })
          .then(response => {
            console.log('New user created successfully:', response.data);
            this.fetchUsers();
            this.closeModal();
          })
          .catch(error => {
            console.error('Error creating user:', error);
          });
    }
    }
  },
  mounted() {
    this.fetchUsers();
  },
};

</script>
<style scoped>
*{
  padding: 0;
  margin: 0;
  box-sizing: border-box;
}
 .add-botton {
  width: 90%;
  display: flex;
  justify-content: end;
}
.content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}
.styled-table {
  width: 80%;
  border-collapse: collapse;
  margin: 25px 0;
  font-size: 18px;
  text-align: center;
  box-shadow: 0 0 20px rgba(0, 0, 0, 0.15);
}

.styled-table thead tr {
  background-color: #333;
  color: #ffffff;
  text-align: center;
}

.styled-table th,
.styled-table td {
  padding: 12px 15px;
}

.styled-table tbody tr {
  border-bottom: 1px solid #dddddd;
}

.styled-table tbody tr:nth-of-type(even) {
  background-color: #f3f3f3;
}

.styled-table tbody tr:last-of-type {
  border-bottom: 2px solid #009879;
}

.styled-table tbody tr:hover {
  background-color: #f1f1f1;
}
.pagination {
  display: flex;
  justify-content: center;
  align-items: center;
  margin-top: 15px;
  gap: 10px;
}

.pagination button {
  padding: 8px 16px;
  font-size: 16px;
  border: none;
  background-color: #009879;
  color: white;
  cursor: pointer;
  border-radius: 4px;
}

.pagination button:disabled {
  background-color: #cccccc;
  cursor: not-allowed;
}

.pagination span {
  font-size: 16px;
}
.table-button {
  display: flex;
}
.table-button button {
  margin: 0 5px;
  border: none;
  border-radius: 5px;
  font-weight: 700;
  cursor: pointer;
}
.table-button button:first-child {
  background: #009879;
  color: white;
  padding: 10px;
}
.table-button button:last-child {
  background: red;
  color: white;
  padding: 10px;
}
.btn-success {
  padding: 10px 20px;
  background-color: #2d73e1;
  color: white;
  border: none;
  border-radius: 5px;
  font-weight: bold;
  cursor: pointer;
}

.btn-success:hover {
  opacity: 0.4;
  color: black;
}

.btn-success:focus {
  outline: none;
  box-shadow: 0 0 3px 2px rgba(40, 167, 69, 0.5);
}
@media(max-width: 830px) {
 h1{
   font-size: 20px;
   margin-bottom: 20px;
 }
  .add-botton {
    display: flex;
    justify-content: end;
  }
  .content {
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
  }
  .styled-table{
    display: flex;
    font-size: 15px;
    margin: 12px 0;
  }
  thead tr{
    display: flex;
    flex-direction: column;
    height: 100%;
    justify-content: center;
  }
  tbody td {
    display: flex;
    flex-direction: column;
    align-items: center;
    flex-wrap: nowrap;
    justify-content: center;
  }
  .table-button {
    display: flex;
    flex-direction: row;
  }
  .table-button button {
    margin: 0 5px;
    border: none;
    border-radius: 5px;
    font-weight: 700;
    cursor: pointer;
  }
  .styled-table tbody {
    display: flex;
    overflow: scroll;
  }
  .styled-table th, .styled-table tbody td {
    padding: 16px 21px;
  }
  .btn-success {
    padding: 6px 10px;
  }

  .pagination button {
    padding: 5px;
  }
}
</style>
