<template>
  <v-container fluid>
    <!-- Alert message component -->
    <v-snackbar v-model="snackVisibility">
      {{ messageAlert }}
      <template v-slot:action="{ attrs }">
        <v-btn
          color="blue"
          text
          v-bind="attrs"
          @click="snackVisibility = false"
        >
          Close
        </v-btn>
      </template>
    </v-snackbar>
    <v-tabs color="deep-purple accent-4" v-model="tab">
      <v-tab>COMIPEMS</v-tab>
      <v-tab>Universidad</v-tab>
    </v-tabs>

    <v-tabs-items v-model="tab">
      <v-tab-item v-if="comipemsData">
        <exam
          examType="comipems"
          :exam="comipemsData.exam"
          v-if="!notComipemsExam"
        />
        <not-pending-exam v-else />
      </v-tab-item>
      <v-tab-item v-if="universidadData">
        <exam
          v-if="!notUniversidadExam"
          examType="universidad"
          :exam="universidadData.exam"
        />
        <not-pending-exam v-else />
      </v-tab-item>
    </v-tabs-items>
  </v-container>
</template>

<script>
import Exam from "@/components/Exam.vue";
import NotPendingExam from "@/components/NotPendingExam.vue";
import { getExamFromDB as getExam } from "@/network/exam";
import { getCurrentExamIDs as getExamIDs } from "@/network/exam";
export default {
  name: "Home",
  components: {
    Exam,
    NotPendingExam,
  },
  data: () => ({
    tab: null,
    snackVisibility: false,
    messageAlert: "",
    comipemsData: null,
    universidadData: null,
    notComipemsExam: false,
    notUniversidadExam: false,
  }),

  methods: {
    async initializeData(type, examID) {
      // Exam query can fail if none matching ID is found
      const exam = await getExam({ type, examID }).catch(() => {
        if (type == "comipems") {
          this.notComipemsExam = true;
        } else if (type == "universidad") {
          this.notUniversidadExam = true;
        }
      });
      const data = { exam };
      return data;
    },
  },
  async mounted() {
    // Get exam IDs
    const ids = await getExamIDs();
    this.comipemsData = await this.initializeData(
      "comipems",
      ids.comipemsExamID
    );
    this.universidadData = await this.initializeData(
      "universidad",
      ids.universidadExamID
    );
  },
};
</script>
