<template>
   <q-dialog ref="createPage" @hide="onDialogHide">
      <q-card>
         <q-stepper
            animated
            color="primary"
            ref="stepper"
            v-model="step"
            alternative-labels
            style="min-width: 320px;"
         >
            <q-step
               icon="folder_open"
               :name="1"
               title="Placement"
               :done="step > 1"
            >
               <div class="column q-gutter-md">
                  <div class="row items-center q-gutter-md">
                     <div class="text-grey-9">Parent Page</div>
                     <q-select
                        color="primary"
                        emit-value
                        filled
                        map-options
                        :options="parentOptions"
                        outline
                        v-model="parentSelected"
                     />
                  </div>

                  <div class="row items-center q-gutter-md">
                     <div class="text-grey-9">Space</div>
                     <q-select
                        color="primary"
                        emit-value
                        filled
                        map-options
                        :options="spaceOptions"
                        outline
                        v-model="spaceSelected"
                     />
                  </div>
               </div>

            </q-step>

            <q-step
               icon="file_copy"
               :name="2"
               title="Template"
               :done="step > 2"
            >
               <page-templates :template.sync="template"/>
            </q-step>

            <q-step
               icon="description"
               title="Details"
               :name="3"
            >
               <page-form :onAdd="onAdd"/>
            </q-step>

            <template v-slot:navigation>
               <q-stepper-navigation>
                  <div class="row justify-end q-gutter-sm">
                     <q-btn
                        v-if="step === 1"
                        color="primary"
                        flat
                        label="Cancel"
                        @click="onCancelClick"
                     />
                     <q-btn
                        v-if="step > 1"
                        color="primary"
                        flat
                        label="Back"
                        @click="$refs.stepper.previous()"
                     />
                     <q-btn
                        v-if="step < 3"
                        color="primary"
                        label="Next"
                        @click="$refs.stepper.next()"
                     />
                  </div>
               </q-stepper-navigation>
            </template>
         </q-stepper>
      </q-card>
   </q-dialog>
</template>

<script>
import { mapActions, mapGetters } from 'vuex'
import { Notify } from 'quasar'

export default {
   data() {
      return {
         parentOptions: [],
         parentSelected: 0,
         spaceOptions: [],
         spaceSelected: null,
         step: 1,
         template: {},
      }
   },

   computed: {
      ...mapGetters([
         'spaces/sorted',
         'pages/sorted'
      ]),

   },

   methods: {
      ...mapActions('pages', ['addPage']),

      async initData() {
         let pageId = this.$route.params.pageId
         let spaceId = this.$route.params.spaceId

         let pages = [...this['pages/sorted'](spaceId)]
         let spaces = [...this['spaces/sorted']]

         let parentOptions = [{ label: 'TOP', value: 0 }]
         let spaceOptions =[]

         pages.map(page => {
            let option = {
               label: page.name,
               value: page.id
            }
            parentOptions.push(option)
         })
         this.parentOptions = parentOptions

         spaces.map(space => {
            if (!space.authorizations.admin && !space.authorizations.create) {
               return false
            }
            let option = {
               label: space.name,
               value: space.id
            }
            spaceOptions.push(option)
            if (space.id === spaceId) {
               this.spaceSelected = space.id
            }
         })
         this.spaceOptions = spaceOptions

         this.parentSelected = pageId ? pageId : parentOptions[0].value
         if (!this.spaceSelected) this.spaceSelected = spaceOptions[0].value
      },

      async onAdd(formData) {
         let spaceId = this.spaceSelected
         formData.spaceId = spaceId
         formData.type = this.template.type
         formData.parent = this.parentSelected
         await this.addPage(formData)
         .then(result => {
            Notify.create('Page added')
            this.$router.push({ name: 'page', params: { spaceId, pageId: result.id } })
         })
         .catch(err => {
            console.error("Error occurred in Dialog", err)
            Notify.create('Error: ' + err)
         })
         .finally(() => {
            this.$emit('hide')
         })
      },

      show() {
         this.$refs.createPage.show()
      },

      hide() {
         this.$refs.createPage.hide()
      },

      onDialogHide() {
         this.$emit('hide')
      },

      onCancelClick() {
         this.hide()
      }
   },

   components: {
      PageTemplates: require("./Templates.vue").default,
      PageForm: require("./Form.vue").default,
   },

   beforeMount() {
      this.initData()
   }
}
</script>

<style scoped>

</style>
