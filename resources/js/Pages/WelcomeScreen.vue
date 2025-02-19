
<template>
  <div v-if="isUpsellUser" class="min-h-screen bg-gradient-to-br from-gray-50 to-gray-100 dark:from-gray-900 dark:to-gray-800">
    <div class="max-w-6xl mx-auto px-4 py-12">
      <div class="bg-white/80 dark:bg-gray-800/80 backdrop-blur-lg rounded-2xl shadow-xl p-8">
        <!-- Progress Steps -->
        <div class="flex justify-between mb-12">
          <template v-for="(section, index) in sections" :key="index">
            <div class="flex items-center" :class="{ 'flex-1': index < sections.length - 1 }">
              <div
                class="w-8 h-8 rounded-full flex items-center justify-center"
                :class="section.isCompleted ? 'bg-green-500 text-white' : 'bg-gray-200 dark:bg-gray-700'"
              >
                <template v-if="section.isCompleted">
                  <svg class="w-5 h-5" fill="none" stroke="currentColor" viewBox="0 0 24 24">
                    <path stroke-linecap="round" stroke-linejoin="round" stroke-width="2" d="M5 13l4 4L19 7" />
                  </svg>
                </template>
                <template v-else>{{ index + 1 }}</template>
              </div>
              <div
                v-if="index < sections.length - 1"
                class="h-1 flex-1 mx-2"
                :class="section.isCompleted ? 'bg-green-500' : 'bg-gray-200 dark:bg-gray-700'"
              ></div>
            </div>
          </template>
        </div>

        <!-- Content Section -->
        <transition name="fade" mode="out-in">
          <div :key="activeSection" class="space-y-8">
            <!-- Section Header -->
            <div class="text-center">
              <h2 class="text-3xl font-bold mb-4">{{ sections[activeSection].title }}</h2>
              <p class="text-gray-600 dark:text-gray-300">{{ sections[activeSection].description }}</p>
            </div>

            <!-- Brand Colors Section -->
            <div v-if="activeSection === 1" class="grid grid-cols-1 md:grid-cols-2 gap-8">
              <div class="space-y-4">
                <label class="block text-sm font-medium">Primary Color</label>
                <input
                  v-model="formData.primaryColor"
                  type="color"
                  class="w-full h-12 rounded-lg cursor-pointer"
                  @change="saveFormData"
                />
              </div>
              <div class="space-y-4">
                <label class="block text-sm font-medium">Secondary Color</label>
                <input
                  v-model="formData.secondaryColor"
                  type="color"
                  class="w-full h-12 rounded-lg cursor-pointer"
                  @change="saveFormData"
                />
              </div>
            </div>

            <!-- Facebook Integration Section -->
            <div v-if="activeSection === 2" class="flex justify-center">
              <button
                @click="connectFacebook"
                class="flex items-center space-x-2 bg-[#1877F2] text-white px-6 py-3 rounded-lg hover:bg-[#1869DB] transition-colors"
              >
                <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24">
                  <path d="M24 12.073c0-6.627-5.373-12-12-12s-12 5.373-12 12c0 5.99 4.388 10.954 10.125 11.854v-8.385H7.078v-3.47h3.047V9.43c0-3.007 1.792-4.669 4.533-4.669 1.312 0 2.686.235 2.686.235v2.953H15.83c-1.491 0-1.956.925-1.956 1.874v2.25h3.328l-.532 3.47h-2.796v8.385C19.612 23.027 24 18.062 24 12.073z"/>
                </svg>
                <span>Connect with Facebook</span>
              </button>
            </div>

            <!-- Terms Section -->
            <div v-if="activeSection === 3" class="space-y-4">
              <div class="p-4 bg-gray-50 dark:bg-gray-900 rounded-lg max-h-60 overflow-y-auto">
                <h3 class="font-medium mb-2">Terms of Service</h3>
                <p class="text-sm text-gray-600 dark:text-gray-400">
                  By using our service, you agree to our terms and conditions regarding the automated creation
                  and posting of advertisements. We ensure compliance with platform-specific guidelines and
                  maintain the highest standards of quality for your brand.
                </p>
              </div>
              <label class="flex items-center space-x-2">
                <input
                  v-model="formData.termsAccepted"
                  type="checkbox"
                  class="rounded border-gray-300"
                  @change="saveFormData"
                />
                <span class="text-sm">I agree to the terms and conditions</span>
              </label>
            </div>

            <!-- Navigation Buttons -->
            <div class="flex justify-between pt-8">
              <button
                @click="previousSection"
                class="px-6 py-2 rounded-lg border border-gray-200 hover:bg-gray-50 transition-colors"
                :class="{ 'invisible': activeSection === 0 }"
              >
                Back
              </button>
              <button
                @click="nextSection"
                class="px-6 py-2 rounded-lg bg-gradient-to-r from-indigo-500 to-purple-500 text-white hover:from-indigo-600 hover:to-purple-600 transition-colors"
              >
                {{ activeSection === sections.length - 1 ? 'Finish' : 'Next' }}
              </button>
            </div>
          </div>
        </transition>
      </div>
    </div>
  </div>
</template>

<script>
import { defineComponent } from 'vue'

export default defineComponent({
  name: 'WelcomeScreen',
  
  data() {
    return {
      isUpsellUser: false,
      activeSection: 0,
      formData: {
        primaryColor: '#000000',
        secondaryColor: '#000000',
        termsAccepted: false,
        facebookAuthorized: false
      },
      sections: [
        {
          title: "Welcome to Animated Ads Galaxy",
          description: "Let's personalize your experience with a few quick steps",
          isCompleted: false
        },
        {
          title: "Brand Colors",
          description: "Choose your brand colors for personalized animations",
          isCompleted: false
        },
        {
          title: "Social Integration",
          description: "Connect with Facebook to automate your social media posts",
          isCompleted: false
        },
        {
          title: "Terms & Conditions",
          description: "Quick review of our terms of service",
          isCompleted: false
        }
      ]
    }
  },

  created() {
    const urlParams = new URLSearchParams(window.location.search)
    this.isUpsellUser = urlParams.get('is_upsell') === '1'
    
    if (this.isUpsellUser) {
      this.loadSavedData()
    }
  },

  methods: {
    loadSavedData() {
      const savedData = localStorage.getItem('onboardingData')
      if (savedData) {
        this.formData = JSON.parse(savedData)
        this.updateSectionCompletion()
      }
    },

    saveFormData() {
      localStorage.setItem('onboardingData', JSON.stringify(this.formData))
      this.updateSectionCompletion()
    },

    updateSectionCompletion() {
      this.sections[0].isCompleted = true
      this.sections[1].isCompleted = !!(this.formData.primaryColor && this.formData.secondaryColor)
      this.sections[2].isCompleted = this.formData.facebookAuthorized
      this.sections[3].isCompleted = this.formData.termsAccepted
    },

    async connectFacebook() {
      try {
        // Here you would implement your Facebook OAuth flow
        // For now, we'll simulate a successful connection
        this.formData.facebookAuthorized = true
        this.saveFormData()
      } catch (error) {
        console.error('Facebook connection failed:', error)
      }
    },

    previousSection() {
      if (this.activeSection > 0) {
        this.activeSection--
      }
    },

    nextSection() {
      if (this.activeSection < this.sections.length - 1) {
        this.activeSection++
      } else {
        // Handle completion - you might want to redirect or show a success message
        this.$inertia.visit('/dashboard')
      }
    }
  }
})
</script>

<style scoped>
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
