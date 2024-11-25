<template>
  <BaseView :title="'SalarySync PAYE Calculator'">
    <v-card>
      <v-card-text>
        <!-- Form Section -->
        <v-form ref="form" v-model="valid">
          <NumberFormatter
            v-model="grossSalary"
            label="Gross Salary (KSh)"
            :rules="[(v) => !!v || 'Salary is required']"
            required
            @keydown.enter="handleEnterKey"
          ></NumberFormatter>

          <!-- Calculate Button -->
          <v-btn
            @click="calculateDeductions"
            :disabled="!valid"
            color="primary"
            block
          >
            Calculate
          </v-btn>
        </v-form>

        <v-divider class="my-4"></v-divider>

        <!-- Results Section -->
        <div v-if="results">
          <v-subheader class="mt-0">Results</v-subheader>
          <v-row>
            <v-col cols="12" md="6">
              <v-list>
                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title
                      ><strong>Gross Salary:</strong></v-list-item-title
                    >
                    <v-list-item-subtitle
                      >KSh
                      {{
                        parseFloat(grossSalary).toLocaleString()
                      }}</v-list-item-subtitle
                    >
                  </v-list-item-content>
                </v-list-item>

                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title
                      ><strong>P.A.Y.E:</strong></v-list-item-title
                    >
                    <v-list-item-subtitle
                      >KSh
                      {{ results.paye.toLocaleString() }}</v-list-item-subtitle
                    >
                  </v-list-item-content>
                </v-list-item>

                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title
                      ><strong>NHIF:</strong></v-list-item-title
                    >
                    <v-list-item-subtitle
                      >KSh
                      {{ results.nhif.toLocaleString() }}</v-list-item-subtitle
                    >
                  </v-list-item-content>
                </v-list-item>

                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title
                      ><strong>Housing Levy:</strong></v-list-item-title
                    >
                    <v-list-item-subtitle
                      >KSh
                      {{
                        results.housingLevy.toLocaleString()
                      }}</v-list-item-subtitle
                    >
                  </v-list-item-content>
                </v-list-item>

                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title
                      ><strong>NSSF:</strong></v-list-item-title
                    >
                    <v-list-item-subtitle
                      >KSh
                      {{ results.nssf.toLocaleString() }}</v-list-item-subtitle
                    >
                  </v-list-item-content>
                </v-list-item>

                <v-list-item>
                  <v-list-item-content>
                    <v-list-item-title
                      ><strong>Net Salary:</strong></v-list-item-title
                    >
                    <v-list-item-subtitle
                      >KSh
                      {{
                        results.netSalary.toLocaleString()
                      }}</v-list-item-subtitle
                    >
                  </v-list-item-content>
                </v-list-item>
              </v-list>
            </v-col>
          </v-row>
        </div>
      </v-card-text>
    </v-card>
  </BaseView>
</template>

<script setup>
import { ref } from "vue";
import BaseView from "@/components/containers/BaseView.vue";
import NumberFormatter from "@/components/NumberFormatter.vue";

const grossSalary = ref(null);
const valid = ref(false);
const results = ref(null);

// NHIF Deduction Rates
const calculateNHIF = (salary) => {
  const nhifBands = [
    { max: 5999, rate: 150 },
    { max: 7999, rate: 300 },
    { max: 11999, rate: 400 },
    { max: 14999, rate: 500 },
    { max: 19999, rate: 600 },
    { max: 24999, rate: 750 },
    { max: 29999, rate: 850 },
    { max: 34999, rate: 900 },
    { max: 39999, rate: 950 },
    { max: 44999, rate: 1000 },
    { max: 49999, rate: 1100 },
    { max: 59999, rate: 1200 },
    { max: 69999, rate: 1300 },
    { max: 79999, rate: 1400 },
    { max: 89999, rate: 1500 },
    { max: 99999, rate: 1600 },
    { max: Infinity, rate: 1700 },
  ];
  return nhifBands.find((band) => salary <= band.max).rate;
};

// PAYE Calculation
const calculatePAYE = (salary) => {
  let paye = 0;
  if (salary <= 24000) {
    paye = salary * 0.1;
  } else if (salary <= 32333) {
    paye = 2400 + (salary - 24000) * 0.25;
  } else {
    paye = 5358.25 + (salary - 32333) * 0.35;
  }
  return paye;
};

// Housing Levy Calculation
const calculateHousingLevy = (salary) => salary * 0.015;

// NSSF Deduction
const calculateNSSF = (salary) => Math.min(salary * 0.06, 2160);

// Main Calculation
const calculateDeductions = () => {
  const gross = parseFloat(grossSalary.value);
  const nhif = calculateNHIF(gross);
  const paye = calculatePAYE(gross);
  const housingLevy = calculateHousingLevy(gross);
  const nssf = calculateNSSF(gross);

  const netSalary = gross - (paye + nhif + housingLevy + nssf);

  results.value = { nhif, paye, housingLevy, nssf, netSalary };
};

// Prevent form submission on Enter
const handleEnterKey = (event) => {
  event.preventDefault();
  calculateDeductions();
};
</script>
<style scoped>
.v-card {
  max-width: 600px;
  margin: 2rem auto;
  border-radius: 12px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.v-btn {
  margin-top: 1rem;
  font-weight: 600;
  border-radius: 8px;
}

.v-subheader {
  font-size: 1.2rem;
  font-weight: bold;
  color: #000700;
}

.v-list-item-title {
  font-size: 1.1rem;
}

.v-list-item-subtitle {
  font-size: 1rem;
  font-weight: normal;
  color: #0b0101;
}

.v-divider {
  border-top: 3px solid #45a948;
}

.v-container {
  padding-top: 2rem;
}
</style>
