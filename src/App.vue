<template>
  <div>
    <el-popover
      placement="bottom"
      title="New Etudiant"
      width="200"
      trigger="click"
    >
      <el-input
        placeholder="Name"
        v-model="name"
        @blur="createEtudiant(name, date)"
      ></el-input>
      <el-button  slot="reference" type="success" style="background-color:blue;"
        >Add New Etudiant</el-button
      >
    </el-popover>
    <el-table
      :data="
        etudiantsData.filter(
          (data) =>
            !search || data.name.toLowerCase().includes(search.toLowerCase())
        )
      "
      style="width: 100%;"
    >
      <el-table-column label="Date" prop="date"> </el-table-column>
      <el-table-column label="Name" prop="name"> </el-table-column>
      <el-table-column align="right">
        <template slot="header" :slot-scope="scope">
          <el-input v-model="search" size="mini" placeholder="Type to search" />
        </template>
        <template slot-scope="scope">
          <el-popover
            placement="bottom"
            title="Edit Etudiant"
            width="200"
            trigger="click"
          >
            <el-input
              placeholder="Name"
              v-model="scope.row.name"
              @blur="updateEtudiant(scope.row.id, scope.row.name, date)"
            ></el-input>
            <el-button size="mini" slot="reference">Edit</el-button>
          </el-popover>
          <el-button
            size="mini"
            style="background-color:red;"
            @click="deleteEtudiant(scope.row.id)"
            >Delete</el-button
          >
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import firebase from "./firebaseInit";
const db = firebase.firestore();

export default {
  data() {
    return {
      name: "",
      date: new Date().toISOString().slice(0, 10),
      etudiantsData: [],
      search: "",
    };
  },
  methods: {
    createEtudiant(name, date) {
      if (name != "") {
        db.collection("etudiants")
          .add({ date: date, name: name })
          .then(() => {
            console.log("Document successfully written!");
            this.readEtudiants();
          })
          .catch((error) => {
            console.error("Error writing document: ", error);
          });
        this.name = "";
      }
    },
    readEtudiants() {
      this.etudiantsData = [];
      db.collection("etudiants")
        .get()
        .then((querySnapshot) => {
          querySnapshot.forEach((doc) => {
            this.etudiantsData.push({
              id: doc.id,
              name: doc.data().name,
              date: doc.data().date,
            });
            console.log(doc.id, " => ", doc.data());
          });
        })
        .catch((error) => {
          console.log("Error getting documents: ", error);
        });
    },
    updateEtudiant(id, name, date) {
      db.collection("etudiants")
        .doc(id)
        .update({
          name: name,
          date: date,
        })
        .then(() => {
          console.log("Document successfully updated!");
          this.readEtudiants();
        })
        .catch((error) => {
          // The document probably doesn't exist.
          console.error("Error updating document: ", error);
        });
    },
    deleteEtudiant(id) {
      db.collection("etudiants")
        .doc(id)
        .delete()
        .then(() => {
          console.log("Document successfully deleted!");
          this.readEtudiants();
        })
        .catch((error) => {
          console.error("Error removing document: ", error);
        });
    },
  },
  mounted() {
    this.readEtudiants();
  },
};
</script>
