<template>
  <div class="container">
    <div >
      <h1 v-if="choice === 0" >CAEN</h1>
      <h1 v-if="choice === 1" >Poste</h1>
      <h1 v-if="choice === 2" >Immatriculation</h1>
      <h1 v-if="choice === 3" >Type de véhicule</h1>
      <h1 v-if="reponse.type <= 4 && choiceVL === 1" >Origine</h1>
      <h1 v-if="reponse.type <= 4 && choiceVL === 2" >Motif origine</h1>
      <h1 v-if="reponse.type <= 4 && choiceVL === 3" >Destination</h1>
      <h1 v-if="reponse.type <= 4 && choiceVL === 4" >Motif destination</h1>
      <h1 v-if="reponse.type <= 4 && choiceVL === 5" >Fréquence du déplacement</h1>
    </div>
  <div v-if="!start">
      <button @click="startSurvey" class="btn-fin">COMMENCER QUESTIONNAIRE</button>
  </div>
    <form v-else @submit.prevent="submitSurvey">
      <div v-if="!showSecondSet">
        <div v-if="choice === 0" class="form-group">
          <input class="form-control" type="text" v-model="reponse.name" placeholder="Prenom enqueteur"
            @keydown.enter.prevent />
          <button v-if="reponse.name.length >= 3" @click="next" class="btn-submit">Suivant</button>
        </div>

        <div v-if="choice === 1" class="form-group">
          <select id="poste" v-model="reponse.poste" class="form-control">
            <option v-for="option in postes" :key="option.id" :value="option.output">
              {{ option.text }}
            </option>
          </select>
          <button v-if="reponse.poste" @click="next" class="btn-submit">Suivant</button>
          <button @click="back" class="btn-return">retour</button>
        </div>

        <div v-if="choice === 2" class="form-group">
          <input class="form-control" type="text" v-model="reponse.plaque" placeholder="Plaque avant"
            @keydown.enter.prevent />
          <button v-if="reponse.plaque" @click="next" class="btn-submit">Suivant</button>
          <button @click="back" class="btn-return">retour</button>
        </div>

        <div v-if="choice === 3" class="form-group">
          <select id="type" v-model="reponse.type" class="form-control">
            <option v-for="option in typeVehicule" :key="option.id" :value="option.output">
              {{ option.text }}
            </option>
          </select>
          <!-- button trigger showSecondSet-->
          <button v-if="reponse.type" @click="buttonSecondSet" class="btn-submit">Suivant</button>
          <button @click="back" class="btn-return">retour</button>
        </div>
      </div>

      <div v-else>
        <div v-if="reponse.type <= 4">
          <div v-if="choiceVL === 0" class="form-group">
      <h1 >Nombre d'occupants</h1>
            <select id="type" v-model="reponse.occupation" class="form-control">
              <option v-for="option in occupation" :key="option.id" :value="option.output">
                {{ option.text }}
              </option>
            </select>
            <button v-if="reponse.occupation" @click="nextVL" class="btn-submit">Suivant</button>
            <button @click="backVL1" class="btn-return">retour</button>
          </div>

          <div v-if="choiceVL === 1">
            <CommuneSelector v-model="reponse.origine" />
            <button v-if="reponse.origine" @click="nextVL" class="btn-submit">Suivant</button>
            <button @click="backVL" class="btn-return">retour</button>
          </div>


          <div v-if="choiceVL === 2" class="form-group">
            <select id="type" v-model="reponse.motifOrigine" class="form-control">
              <option v-for="option in motifOrigine" :key="option.id" :value="option.output">
                {{ option.text }}
              </option>
            </select>
            <button v-if="reponse.motifOrigine" @click="nextVL" class="btn-submit">Suivant</button>
            <button @click="backVL" class="btn-return">retour</button>
          </div>

          <div v-if="choiceVL === 3">
            <CommuneSelector v-model="reponse.destination" />
            <button v-if="reponse.destination" @click="nextVL" class="btn-submit">Suivant</button>
            <button @click="backVL" class="btn-return">retour</button>
          </div>

          <div v-if="choiceVL === 4" class="form-group">
            <select id="type" v-model="reponse.motifDestination" class="form-control">
              <option v-for="option in motifDestination" :key="option.id" :value="option.output">
                {{ option.text }}
              </option>
            </select>
            <button v-if="reponse.motifDestination" @click="nextVL" class="btn-submit">Suivant</button>
            <button @click="backVL" class="btn-return">retour</button>
          </div>

          <div v-if="choiceVL === 5" class="form-group">
            <select id="type" v-model="reponse.frequence" class="form-control">
              <option v-for="option in frequence" :key="option.id" :value="option.output">
                {{ option.text }}
              </option>
            </select>
            <button @click="backVL" class="btn-return">retour</button>
          </div>
        </div>
        <div v-else-if="reponse.occupation > 4">PL</div>
      </div>

      <input v-show="allFieldsFilled" type="submit" value="Terminer" class="btn-fin" :disabled="isSubmitDisabled" />
    </form>
  </div>
  <button @click="downloadData" class="btn-data">Download Data</button>
</template>

<script setup>
import { ref, computed } from "vue";
import CommuneSelector from './CommuneSelector.vue';
import { postes, typeVehicule, occupation, motifOrigine, motifDestination, frequence } from "./reponses";
import { db } from "../../firebaseConfig";
import { collection, addDoc, getDocs } from "firebase/firestore";
import * as XLSX from "xlsx";


const surveyCollectionRef = collection(db, "Caen");
const start = ref(false);
const startDate = ref('')
const num = ref(1);
const choice = ref(0);
const choiceVL = ref(0);
const showSecondSet = ref(false);
const reponse = ref({
  name: '',
  poste: "",
  plaque: "",
  type: 0,
  num: num.value,
  occupation: "",
  origine: "",
  motifOrigine: "",
  destination: "",
  motifDestination: "",
  frequence: ""
});



const startSurvey = () => {
  start.value = true;
  startDate.value = new Date().toLocaleTimeString("fr-FR").slice(0, 8);
}

const buttonSecondSet = () => {
  showSecondSet.value = true;
  choice.value++;
}



const next = () => {
  choice.value++;
};

const back = () => {
  choice.value--;
};

const nextVL = () => {
  choiceVL.value++;
};

const backVL = () => {
  choiceVL.value--;
};

const backVL1 = () => {
  reponse.value.type = ""
}

const allFieldsFilled = computed(() => {
  return (
    reponse.value.poste !== "" &&
    reponse.value.plaque !== "" &&
    reponse.value.type !== "" &&
    reponse.value.occupation !== "" &&
    reponse.value.origine !== "" &&
    reponse.value.motifOrigine !== "" &&
    reponse.value.destination !== "" &&
    reponse.value.motifDestination !== "" &&
    reponse.value.frequence !== ""
  );
});




const isSubmitDisabled = computed(() => {
  return (
    reponse.value.name === "" ||
    reponse.value.poste === "" ||
    reponse.value.plaque === "" ||
    reponse.value.type === "" ||
    reponse.value.origine === "" ||
    reponse.value.occupation === "" ||
    reponse.value.motifOrigine === "" ||
    reponse.value.destination === "" ||
    reponse.value.motifDestination === "" ||
    reponse.value.frequence === ""
  );
});

const submitSurvey = async () => {
  await addDoc(surveyCollectionRef, {
    q1: reponse.value.poste,
    q2: new Date().toLocaleDateString("fr-FR").replace(/\//g, "-"),
    q4: reponse.value.name,
    q5: new Date().toLocaleTimeString("fr-FR").slice(0, 8),
    q6: reponse.value.plaque,
    q7: reponse.value.type,
    q8: reponse.value.occupation,
    q9: reponse.value.origine,
    q10: reponse.value.motifOrigine,
    q11: reponse.value.destination,
    q12: reponse.value.motifDestination,
    q13: reponse.value.frequence,
  });
  choice.value = 0;
  choiceVL.value = 0;
  reponse.value.name = "";
  reponse.value.poste = "";
  reponse.value.plaque = "";
  reponse.value.type = "";
  reponse.value.occupation = "";
  reponse.value.origine = "";
  reponse.value.motifOrigine = "";
  reponse.value.destination = "";
  reponse.value.motifDestination = "";
  reponse.value.frequence = "";
  showSecondSet.value = false;
  start.value = false;

};

const downloadData = async () => {
  try {
    const querySnapshot = await getDocs(surveyCollectionRef);
    let data = [];
    let maxWidths = {}; // Object to keep track of maximum width for each column
    const minWidth = 3; // Minimum width in Excel units,

    querySnapshot.forEach((doc) => {
      let docData = doc.data();
      let mappedData = {
        q1: docData.q1 || "",
        q2: docData.q2 || "",
        q3: doc.id, // Firebase document ID
        name: docData.q4 || "",
        q5: docData.q5 || "",
        q6: docData.q6 || "",
        q7: docData.q7 || "",
        q8: docData.q8 || "",
        q9: docData.q9 || "",
        q10: docData.q10 || "",
        q11: docData.q11 || "",
        q12: docData.q12 || "",
        q13: docData.q13 || "",
      };
      data.push(mappedData);
    });

    data.sort((a, b) => {
      // Assuming q4 is a number. If it's a string or a different type, you might need to modify this comparison
      return a.q4 - b.q4;
    });

    // Calculate the maximum width for each column, considering the minimum width
    const scalingFactor = 1.2; // Adjust this factor as needed for padding or wider characters

    Object.keys(data[0]).forEach((key) => {
      let maxLen = Math.max(
        ...data.map((item) => item[key].toString().length),
        minWidth
      );

      maxWidths[key] = Math.ceil(maxLen * scalingFactor); // Apply scaling factor and round up
    });


    // Convert data to a worksheet
    const worksheet = XLSX.utils.json_to_sheet(data, {
      header: [
        "q1",
        "q2",
        "q3",
        "name",
        "q5",
        "q6",
        "q7",
        "q8",
        "q9",
        "q10",
        "q11",
        "q12",
        "q13"
      ],
      skipHeader: false,
    });

    // Set the widths for each column, ensuring a minimum width
    worksheet["!cols"] = Object.keys(maxWidths).map((key) => ({
      wch: maxWidths[key],
    }));

    const workbook = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(workbook, worksheet, "Data");

    // Export the workbook to a .xlsx file
    XLSX.writeFile(workbook, "OdCaen.xlsx");
  } catch (error) {
    console.error("Error downloading data: ", error);
  }
};
</script>

<style>
body {
  background-color: #1e1e1e;
}

.container {
  background-color: #1e1e1e;
  color: white;
  width: 75%;
  padding: 5% 0;
  margin: auto;
}

label {
  display: block;
  margin-bottom: 5px;
}
.form-group {
  margin-bottom: 15px;
}


.form-control {
  width: 100%;
  padding: 10px !important;
  border-radius: 5px;
  border: 1px solid #333;
  background-color: #333;
  color: white;
  text-transform: uppercase;
}

.form-control, .btn-submit {
  box-sizing: border-box;
}
.btn-submit {
  width: 100%;
  background-color: #4caf50;
  color: white;
  padding: 10px 20px;
  margin-top: 1.5%;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.btn-submit:hover {
  background-color: #45a049;
}

.btn-fin {
  width: 100%;
  background-color: #4c4faf;
  color: white;
  padding: 20px 20px;
  margin-top: 20%;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.btn-fin:hover {
  background-color: #45a049;
}

.btn-return {
  background-color: #898989;
  color: white;
  padding: 10px 20px;
  margin-top: 1.5%;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  width: 100%;
}

.btn-return:hover {
  background-color: #839684;
}

.btn-data {
  position: absolute;
  bottom: 0;
  left: 0;
  background-color: #4caf50;
  color: white;
  padding: 10px 20px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  width: 100%;
}

.btn-data:hover {
  background-color: #45a049;
}

h1 {
  text-align: center;
  color: #4caf50;
}

.commune-dropdown {
  /* Style your dropdown list here */
  list-style-type: none;
  padding: 0;
  margin: 0;
  border: 1px solid #ccc;
  border-radius: 4px;
  max-height: 200px;
  overflow-y: auto;
}

.commune-dropdown li {
  padding: 5px 10px;
  cursor: pointer;
}

.commune-dropdown li:hover {
  background-color: #f0f0f0;
}

input, select, button {
  font-size: 16px;
  padding: 10px ;
}

</style>
