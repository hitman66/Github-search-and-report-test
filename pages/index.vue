<template>
  <v-layout>
    <v-flex class="text-center">
      <template v-if= "errorCheck">
      <v-alert type="error">
        {{this.errorMsg}}
      </v-alert>
      </template>    
      <vuetify-logo />
      <h2> Github Search & Report Web Application </h2>
      <br>
      <template v-if= "(submitType === 'language') || (submitType === 'topic')">
      <v-btn class="mr-4" @click="reportPrint">Generate Report</v-btn> 
      </template>
      <br>
    <form>
      <v-text-field
        v-model="repo"
        ref="repo"
        label="Repository"
      ></v-text-field>
      <v-text-field
        v-model="topic"
        label="Topic"
        ref="topic"
      ></v-text-field>
      <v-text-field
        v-model="programmingLanguage"
        label="Programming Language"
        ref="programmingLanguage"
      ></v-text-field>
      <v-btn class="mr-4" @click="refresh">Refresh</v-btn>
      <v-btn class="mr-4" @click="submitTopic">Search Repository based on topic</v-btn>
      <v-btn class="mr-4" @click="submitLanguage">Search Repository based on languages</v-btn>
    </form>
    <br>
    <template v-if= "(submitType === 'language') || (submitType === 'topic')">
    <v-simple-table dark>
      <template v-slot:default>
        <thead>
          <tr>
            <th class="text-left">Name</th>
            <th class="text-left">Description</th>
            <th class="text-left">Link</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="item in tableData" :key="item.id">
            <td class="text-left">{{ item.name }}</td>
            <td class="text-left">{{ item.description }}</td>
            <td class="text-left">        
              <div class="my-2">
              <v-btn small color="primary" @click="openUrl(`${item.html_url}`)" >Click Here</v-btn>
              </div> 
           </td>
          </tr>
        </tbody>
      </template>
    </v-simple-table>
    </template>
    <br>
    <template v-if= "submitType === 'language'">
    <div class="text-center">
      <v-pagination
        v-model="page.current"
        :length="page.total"
        :total-visible="8"
        @input="fetchLanguage"
      ></v-pagination>
    </div>        
    </template>
    <template v-else-if="submitType === 'topic'">
      <v-pagination
        v-model="page.current"
        :length="page.total"
        :total-visible="8"
        @input="fetchTopic"
      ></v-pagination>
    </div>        
    </template>      
    </v-flex>
  </v-layout>
</template>

<script>

import VuetifyLogo from '~/components/VuetifyLogo.vue'

export default {
  components: {
    VuetifyLogo,
  },
    methods: {     
      async submitTopic() {

          //If cannot find both repo & topic
          if ((!this.repo) && (!this.topic)) {

            this.errorCheck = "repo+topic"
            this.errorMsg = this.errorRepoTopic
            this.$refs.repo.focus()
            return
            
          }

          //If cannot find repo
          else if (!this.repo) {

            this.errorCheck = this.repoText
            this.errorMsg = this.errorRepo
            this.$refs.repo.focus()
            return            

          }

          //If cannot find topic
          else if (!this.topic) {

            this.errorCheck = this.topicText
            this.errorMsg = this.errorTopic
            this.$refs.topic.focus()
            return                       

          }

          //This will pop-up table and pagination UI
          this.submitType = this.topicText
          this.errorCheck = null
          this.errorMsg = null
          this.pageTopic = this.topic
          this.pageRepo = this.repo

          //fill any blank spaces with '+' symbol within the search input
          var repoText = this.repo.split(' ').join('+')
          var topicText = this.topic.split(' ').join('+')

          //Grabbing the data through Github's API v3
          this.grabRepoTopicData(repoText,topicText, 1)
         
      },
      async submitLanguage() {

          //If cannot find both repo & languages
          if ((!this.repo) && (!this.programmingLanguage)) {

            this.errorCheck = "repo+language"
            this.errorMsg = this.errorRepoLanguage
            this.$refs.repo.focus()
            return
            
          }

          //If cannot find repo
          else if (!this.repo) {

            this.errorCheck = this.repoText
            this.errorMsg = this.errorRepo
            this.$refs.repo.focus()
            return            

          }

          //If cannot find language
          else if (!this.programmingLanguage) {

            this.errorCheck = this.languageText
            this.errorMsg = this.errorLanguage
            this.$refs.programmingLanguage.focus()
            return                       

          }

          //This will pop-up table and pagination UI
          this.submitType = this.languageText
          this.errorCheck = null
          this.errorMsg = null
          this.pageLanguage = this.programmingLanguage
          this.pageRepo = this.repo

          //fill any blank spaces with '+' symbol within the search input
          var repoText = this.repo.split(' ').join('+')
          var languageText = this.programmingLanguage.split(' ').join('+')

          //Grabbing the data through Github's API v3
          this.grabRepoLanguageData(repoText,languageText, 1)
        
      },
      openUrl(url) {

         window.open(url, "_blank");

      },
      refresh() {

          this.$refs.repo.reset()
          this.$refs.topic.reset()
          this.$refs.programmingLanguage.reset()
          this.submitType = null
          this.errorCheck = null
          this.errorMsg = null
          this.githubData = null
          this.page.current = 1
          this.page.total = 0
          this.pageTopic = null
          this.pageLanguage = null
          this.pageRepo = null

      },   
      async fetchLanguage() {

      //fill any blank spaces with '+' symbol within the search input
      var repoText = this.pageRepo.split(' ').join('+')
      var languageText = this.pageLanguage.split(' ').join('+')

      //Grabbing the data through Github's API v3 based on what page that being directed to
      this.grabRepoLanguageData(repoText, languageText, this.page.current)

      },
      async fetchTopic() {

      //fill any blank spaces with '+' symbol within the search input
      var repoText = this.pageRepo.split(' ').join('+')
      var topicText = this.pageTopic.split(' ').join('+')

      //Grabbing the data through Github's API v3 based on what page that being directed to
      this.grabRepoTopicData(repoText, topicText, this.page.current)

      },
      async grabRepoTopicData(repo, topic, page) {

          //fetch data from API
          this.githubData = await this.$axios.get('https://api.github.com/search/repositories?q=' + repo + '+topic:' + topic + '&order=desc&page=' + page, 
          { headers: {
            Accept: 'application/vnd.github.mercy-preview+json'
              }
          })

          //populate the table with data obtained from the API
          var tableData = this.githubData.data.items
          this.tableData = tableData
          this.page.total = parseInt((parseInt(this.githubData.data.total_count)/30).toFixed(0))
          if (this.page.total > 1000) {

            this.page.total = parseInt(parseInt(1000/30).toFixed(0))

          }

          return

      },
      async grabRepoLanguageData(repo, language, page) {

          //fetch data from API
          this.githubData = await this.$axios.get('https://api.github.com/search/repositories?q=' + repo + '+language:' + language + '&order=desc&page=' + page)       

          //populate the table with data obtained from the API
          var tableData = this.githubData.data.items
          this.tableData = tableData
          this.page.total = parseInt((parseInt(this.githubData.data.total_count)/30).toFixed(0))
          if (this.page.total > 1000) {

            this.page.total = parseInt(parseInt(1000/30).toFixed(0))

          }

          return

      },           
      fetchPageLanguage() {
          this.fetchLanguage()
      },
      fetchPageTopic() {
          this.fetchTopic()
      },      
      reportPrint() {

        //Using jsPDF to generate PDF file for report
            if (process.client) {
        const jsPDF = require('jspdf');
        require('jspdf-autotable');        
        let doc = new jsPDF();
        let pdfName = 'Search Result Report'
        let tableData = this.githubData.data.items
        var array = []
        var i
        //populate the table inside with the data fetched from the API earlier on the webpage
        for (i = 0; i < tableData.length; i++) {
            array.push([this.githubData.data.items[i].name, this.githubData.data.items[i].description])
        } 

        let repoText = "Repository: " + this.pageRepo
        let languageText = "Programming Language: " + this.pageLanguage
        let topicText = "Topic: " + this.pageTopic
        let countText = "This table shows only " + tableData.length + " from a total of " + this.githubData.data.total_count + " search results"

        doc.setFontSize(30);
        doc.text("Search Report",15 , 20)
        doc.setFontSize(12);
        doc.text(repoText, 15, 40)

        if (this.submitType === 'language') {

        doc.text(languageText, 120, 40)

        }

        else {

        doc.text(topicText, 120, 40)

        }

        doc.autoTable({
          startY: 50,
          theme: 'grid',
          body: array,
          head: [['Name','Description']],
        })
        doc.text(
        countText,
        14,
        doc.autoTable.previous.finalY + 10
        )        
        doc.save(pdfName + '.pdf')
        
    }

      },
    },
    data () {
      return {
        headers: [
          {
            text: 'Name Project',
            align: 'start',
            sortable: false,
            value: 'name',
          },
          { text: 'Description', value: 'description' },
        ],
        repo: null,
        programmingLanguage: null,
        topic: null,
        githubData: [{}],
        tableData: [{}],
        topicText : "topic",
        languageText: "language",
        repoText: "repo",
        errorRepoLanguage: "Please input the repository & programming languages that you want to search.",
        errorRepo: "Please input the repository that you want to search",
        errorRepoTopic: "Please input the repository & topic that you want to search.",
        errorLanguage: "Please input the programming languages that you want to search.",
        errorTopic: "Please input the topic that you want to search.",
        errorCheck: null,
        errorMsg: null,
        submitType: null,
        pageRepo: null,
        pageTopic: null,
        pageLanguage: null,
        page: {
            current: 1,
            total: 0
        },
        language: null,
      }
    },
  }
</script>