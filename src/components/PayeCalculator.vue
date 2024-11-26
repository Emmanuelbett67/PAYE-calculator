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
                      ><strong>NHIF/SHIF:</strong></v-list-item-title
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
import BaseView from "@/components/containers/baseview.vue";
import NumberFormatter from "@/components/NumberFormatter.vue";

const grossSalary = ref(null);
const valid = ref(false);
const results = ref(null);

// NHIF Deduction Rates
const calculateNHIF = (salary) => {
  if (salary <= 5999) return 150;
  if (salary <= 7999) return 300;
  if (salary <= 11999) return 400;
  if (salary <= 14999) return 500;
  if (salary <= 19999) return 600;
  if (salary <= 24999) return 750;
  if (salary <= 29999) return 850;
  if (salary <= 34999) return 900;
  if (salary <= 39999) return 950;
  if (salary <= 44999) return 1000;
  if (salary <= 49999) return 1100; // For salary of 50,000
  return 1200; // For salary above 50,000
};


// PAYE Calculation
const calculatePAYE = (salary) => {
  let paye = 0;
  const personalRelief = 2400; // Fixed personal relief

  if (salary <= 24000) {
    paye = salary * 0.1; // 10% on the first 24,000
  } else if (salary <= 32333) {
    paye = 2400 + (salary - 24000) * 0.25; // 25% on the next 8,333
  } else {
    paye = 2400 + 2083.25 + (salary - 32333) * 0.35; // 35% on the remaining
  }

  // Subtract personal relief from PAYE
  paye = Math.max(0, paye - personalRelief); // Ensure PAYE doesn't go negative

  return parseFloat(paye.toFixed(2)); // Round to 2 decimals
};



// Housing Levy Calculation
const calculateHousingLevy = (salary) => {
  return parseFloat((salary * 0.015).toFixed(2)); // 1.5%
};

// NSSF Deduction
const calculateNSSF = (salary) => {
  const nssfRate = 0.06;
  const nssfMax = 2160; // Capped at 2,160
  return Math.min(salary * nssfRate, nssfMax);
};

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
