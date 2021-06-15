<template>
  <!--
    Add the directive called contactBubble to
    this element
  -->
  <div id="app" v-contactBubble>
    <div class="section">
      <button id="btnAddContact" @click.prevent="openNew">Add Contact</button>
    </div>
    <div class="section">
      <!-- Two way bind query to this input field -->
      <input v-model="query" id="search" placeholder="search" type="text">
    </div>
    <app-contactform v-bind="formSettings" v-bind:contact="contact" v-bind:index="selectedIndex"></app-contactform>
    <!--
      Edit app-contacts component to receive
      contacts property through search results
    -->
    <app-contacts v-bind:contacts="search"></app-contacts>
  </div>
</template>

<script>
import Contacts from './components/Contacts'
import ContactForm from './components/ContactForm'
import { ContactBus } from './bus/ContactBus'
import sampleData from './assets/contacts.json'

export default {
  name: 'App',
  components: {
    'app-contacts': Contacts,
    'app-contactform': ContactForm
  },
  data () {
    /**
     * Create the necessary data variables here
     */
    return {
      selectedIndex: null,
      contacts: [...sampleData],
      contact: {
        firstname: '',
        lastname: '',
        email: [''],
        phoneNumber: ['']
      },
      query: '',
      sortedContacts: [],
      formSettings: {
        /**
         * Create members boolean variables visible and newCon
         * with default values false. Create member variable
         * index with default value ''
         */
        index: '',
        newCon: false,
        visible: false
      }
    }
  },
  computed: {
    search: {
      /**
       * Complete this function to
       * search through contacts
       * by first and last name, email & phone number
       * and return an array with the results
       */
      get: function () {
        let results = this.sortedContacts
        if (this.query) {
          const query = this.query.toLowerCase()
          results = results.filter((item) => item.firstname.toLowerCase().includes(query) || item.lastname.toLowerCase().includes(query) || item.phoneNumber.some(phone => phone.toLowerCase().includes(query)) || item.email.some(email => email.toLowerCase().includes(query)))
        }

        return results
      }
    }
  },
  mounted () {
    this.sortedContacts = this.sortContacts()
  },
  directives: {
    contactBubble: {
      /**
       * Complete this directive
       * to make its elements
       * background-color: rgb(159, 159, 199);
       * border-radius: 20px;
       * color: white;
       * font-variant: all-petite-caps;
       */
      bind (el) {
        el.style.backgroundColor = 'rgb(159, 159, 199)'
        el.style.borderRadius = '20px'
        el.style.color = 'white'
        el.style.fontVariant = 'all-petite-caps'
        el.style.fontSize = '20px'
      }
    }
  },
  methods: {
    sortContacts: function () {
      /**
       * Complete this function to return contacts
       * sorted by lastname in ascending order
       */
      return this.contacts.sort((a, b) => (a.lastname.toLowerCase() > b.lastname.toLowerCase()) ? 1 : -1)
    },
    openNew: function () {
      /**
       * This function should be used to show new contact
       * form on screen.
       */
      this.formSettings.visible = true
      this.formSettings.newCon = true

      this.contact = {
        id: this.contacts.length,
        firstname: '',
        lastname: '',
        email: [''],
        phoneNumber: ['']
      }
    },
    close: function (data) {
      this.formSettings.visible = !data
    }
  },
  beforeCreate () {
    /**
     * Each instance of App will load a separate clone of the mocks contacts data
     */
    const mockContacts = require('./assets/contacts.json')
    const clone = JSON.parse(JSON.stringify(mockContacts))
    this.contacts = clone
  },
  created: function () {
    ContactBus.$on('addCon', (data) => {
      /**
       * Finish this method so that data will
       * be appended to contacts if formSettings.newCon
       * is true and reset formSettings to default values
       */
      this.contacts.push(data)
    })

    ContactBus.$on('noAddCon', (data) => {
      this.nullContact = data
    })

    ContactBus.$on('close', (data) => {
      this.close(data)
    })

    ContactBus.$on('delCon', (index) => {
      /**
       * This method should remove index from contacts
       * and set formSettings to their default values
       */
      const contact = this.sortedContacts[index]
      const contactIndex = this.contacts.findIndex(c => c.id === contact.id)
      this.contacts.splice(contactIndex, 1)
    })

    ContactBus.$on('editCon', (contactIndex) => {
      /**
       * Finish this method to show the form
       * with the index of the contact to be
       * displayed.
       */
      this.formSettings.newCon = false
      this.formSettings.visible = true
      this.selectedIndex = contactIndex
      this.contact = this.sortedContacts[contactIndex]
    })
  }
}

</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
.section {
  margin: 30px;
}
</style>
