<script setup>
import { computed, reactive, ref } from 'vue'

const symptomOptions = [
  'Fever',
  'Cough',
  'Sore throat',
  'Shortness of breath',
  'Nausea',
  'Fatigue',
  'Headache',
  'Dizziness',
  'Body aches',
  'Abdominal pain',
]

const illnessOptions = [
  'Asthma',
  'Diabetes',
  'High blood pressure',
  'Heart condition',
  'Anxiety',
  'Seasonal allergies',
  'Pregnancy',
  'None',
]

const medicationOptions = [
  'Vitamin D',
  'Ibuprofen',
  'Acetaminophen',
  'Antibiotics',
  'Allergy medicine',
  'Blood pressure medication',
  'Insulin',
  'None',
]

const form = reactive({
  firstName: '',
  lastName: '',
  dateOfBirth: '',
  phone: '',
  email: '',
  symptoms: [],
  customSymptoms: '',
  pain: 4,
  preexistingIllnesses: [],
  customHealthHistory: '',
  currentMedications: [],
  insuranceProvider: '',
  memberId: '',
  planType: 'Private insurance',
  comments: '',
})

const errors = reactive({})
const submitted = ref(false)

const painLabel = computed(() => {
  if (form.pain <= 2) return 'Mild discomfort'
  if (form.pain <= 5) return 'Moderate discomfort'
  if (form.pain <= 8) return 'Strong pain'
  return 'Severe pain'
})

const progress = computed(() => {
  const fields = [
    form.firstName,
    form.lastName,
    form.dateOfBirth,
    form.phone,
    form.email,
    form.symptoms.length > 0 || form.customSymptoms.trim(),
    form.pain > 0,
    form.preexistingIllnesses.length > 0 || form.customHealthHistory.trim(),
    form.insuranceProvider,
    form.memberId,
    form.comments.trim(),
  ]

  const completed = fields.filter(Boolean).length
  return Math.round((completed / fields.length) * 100)
})

function toggleChoice(list, value) {
  if (list.includes(value)) {
    list.splice(list.indexOf(value), 1)
  } else {
    list.push(value)
  }
}

function validateForm() {
  Object.keys(errors).forEach((key) => delete errors[key])

  if (!form.firstName.trim()) errors.firstName = 'Please add your first name.'
  if (!form.lastName.trim()) errors.lastName = 'Please add your last name.'
  if (!form.dateOfBirth) errors.dateOfBirth = 'Please share your date of birth.'
  if (!form.phone.trim()) errors.phone = 'Please add a phone number.'
  if (!form.email.trim()) errors.email = 'Please add your email address.'
  if (!form.symptoms.length && !form.customSymptoms.trim()) {
    errors.symptoms = 'Tell us what symptoms you are having.'
  }
  if (!form.insuranceProvider.trim()) errors.insuranceProvider = 'Please add your insurance provider.'
  if (!form.memberId.trim()) errors.memberId = 'Please add your member ID.'

  return Object.keys(errors).length === 0
}

function submitForm() {
  submitted.value = false
  if (!validateForm()) return

  submitted.value = true
}
</script>

<template>
  <div class="app-shell">
    <div class="mobile-frame">
      <header class="topbar">
        <div>
          <p class="brand-kicker">Welcome</p>
          <h1>Harbor Urgent Care</h1>
        </div>
        <button class="ghost-button" type="button">Need help?</button>
      </header>

      <main class="content">
        <section class="intro-card">
          <div class="badge">We’re here for you</div>
          <h2>Let’s get you checked in.</h2>
          <p>
            We know waiting can feel stressful. Share a few details and we’ll take it from here.
          </p>
        </section>

        <section class="progress-card" aria-live="polite">
          <div class="progress-row">
            <span>Check-in progress</span>
            <strong>{{ progress }}%</strong>
          </div>
          <div class="progress-bar">
            <span :style="{ width: `${progress}%` }"></span>
          </div>
        </section>

        <form class="intake-form" @submit.prevent="submitForm">
          <section class="form-panel">
            <div class="section-header">
              <span class="section-index">01</span>
              <h3>Patient details</h3>
            </div>

            <div class="field-grid two-up">
              <label class="field">
                <span>First name</span>
                <input v-model="form.firstName" type="text" placeholder="Jordan" />
                <small v-if="errors.firstName" class="error">{{ errors.firstName }}</small>
              </label>

              <label class="field">
                <span>Last name</span>
                <input v-model="form.lastName" type="text" placeholder="Lee" />
                <small v-if="errors.lastName" class="error">{{ errors.lastName }}</small>
              </label>
            </div>

            <div class="field-grid two-up">
              <label class="field">
                <span>Date of birth</span>
                <input v-model="form.dateOfBirth" type="date" />
                <small v-if="errors.dateOfBirth" class="error">{{ errors.dateOfBirth }}</small>
              </label>

              <label class="field">
                <span>Phone</span>
                <input v-model="form.phone" type="tel" placeholder="(555) 123-4567" />
                <small v-if="errors.phone" class="error">{{ errors.phone }}</small>
              </label>
            </div>

            <label class="field">
              <span>Email</span>
              <input v-model="form.email" type="email" placeholder="name@example.com" />
              <small v-if="errors.email" class="error">{{ errors.email }}</small>
            </label>
          </section>

          <section class="form-panel">
            <div class="section-header">
              <span class="section-index">02</span>
              <h3>Symptoms</h3>
            </div>

            <div class="choice-grid">
              <button
                v-for="symptom in symptomOptions"
                :key="symptom"
                class="choice-chip"
                :class="{ selected: form.symptoms.includes(symptom) }"
                type="button"
                @click="toggleChoice(form.symptoms, symptom)"
              >
                {{ symptom }}
              </button>
            </div>

            <label class="field top-space">
              <span>Other symptoms</span>
              <textarea
                v-model="form.customSymptoms"
                rows="3"
                placeholder="Describe any symptoms not listed above."
              ></textarea>
            </label>
            <small v-if="errors.symptoms" class="error">{{ errors.symptoms }}</small>
          </section>

          <section class="form-panel">
            <div class="section-header">
              <span class="section-index">03</span>
              <h3>Pain level</h3>
            </div>

            <div class="pain-panel">
              <div class="pain-header">
                <span>How bad is it?</span>
                <strong>{{ form.pain }}/10</strong>
              </div>
              <input v-model.number="form.pain" type="range" min="0" max="10" step="1" />
              <div class="pain-scale">
                <span>Zero</span>
                <span>{{ painLabel }}</span>
                <span>Severe</span>
              </div>
            </div>
          </section>

          <section class="form-panel">
            <div class="section-header">
              <span class="section-index">04</span>
              <h3>Health history</h3>
            </div>

            <div class="choice-grid compact">
              <button
                v-for="condition in illnessOptions"
                :key="condition"
                class="choice-chip"
                :class="{ selected: form.preexistingIllnesses.includes(condition) }"
                type="button"
                @click="toggleChoice(form.preexistingIllnesses, condition)"
              >
                {{ condition }}
              </button>
            </div>

            <label class="field top-space">
              <span>Current medications</span>
              <div class="choice-grid compact">
                <button
                  v-for="medication in medicationOptions"
                  :key="medication"
                  class="choice-chip"
                  :class="{ selected: form.currentMedications.includes(medication) }"
                  type="button"
                  @click="toggleChoice(form.currentMedications, medication)"
                >
                  {{ medication }}
                </button>
              </div>
            </label>

            <label class="field top-space">
              <span>Additional health history</span>
              <textarea
                v-model="form.customHealthHistory"
                rows="3"
                placeholder="Share any relevant past conditions, recent illnesses, or medical details."
              ></textarea>
            </label>
          </section>

          <section class="form-panel">
            <div class="section-header">
              <span class="section-index">05</span>
              <h3>Insurance</h3>
            </div>

            <label class="field">
              <span>Insurance provider</span>
              <input v-model="form.insuranceProvider" type="text" placeholder="BlueCross BlueShield" />
              <small v-if="errors.insuranceProvider" class="error">{{ errors.insuranceProvider }}</small>
            </label>

            <div class="field-grid two-up">
              <label class="field">
                <span>Plan type</span>
                <select v-model="form.planType">
                  <option>Private insurance</option>
                  <option>Medicaid</option>
                  <option>Medicare</option>
                  <option>Self-pay</option>
                </select>
              </label>

              <label class="field">
                <span>Member ID</span>
                <input v-model="form.memberId" type="text" placeholder="A12345678" />
                <small v-if="errors.memberId" class="error">{{ errors.memberId }}</small>
              </label>
            </div>
          </section>

          <section class="form-panel">
            <div class="section-header">
              <span class="section-index">06</span>
              <h3>Anything else?</h3>
            </div>

            <label class="field">
              <span>Comments</span>
              <textarea v-model="form.comments" rows="4" placeholder="Tell us what concerns you most today."></textarea>
            </label>
          </section>

          <div v-if="submitted" class="success-banner" role="status">
            Thanks, {{ form.firstName || 'friend' }}. Your details have been captured and our team will review them shortly.
          </div>

          <button class="submit-button" type="submit">Continue to check-in</button>
        </form>
      </main>

      <nav class="floating-nav" aria-label="Quick actions">
        <button type="button" class="nav-pill active">
          <span>💬</span>
          Chat
        </button>
        <button type="button" class="nav-pill">
          <span>❔</span>
          FAQs
        </button>
      </nav>
    </div>
  </div>
</template>
