<!-- This sidebar displays the selected editor and has an own header -->
<template>
  <div
    class="editor-sidebar"
    v-smoothscrollbar="{ listener, options }"
  >
    <EditorHeader
      @close="close"
      @update-date="updateDate"
      :editorTitle="editorType"
    />
    <hr>
    <div v-if="editorId === 0">
      <NoteEditor @create-note="createNote"/>
    </div>
    <div v-else-if="editorId === 1">
      <ImageUpload @upload-image="uploadImage"/>
    </div>
    <div v-else-if="editorId === 2">
      <FileUpload @upload-file="uploadFile"/>
    </div>
    <div v-else-if="editorId === 3">
      <PollEditor @create-poll="createPoll"/>
    </div>
    <div v-else-if="editorId === 4">
      <SurveyEditor @create-survey="createSurvey"/>
    </div>
    <br><br><br><br> <!-- For scrollbar -->
  </div>
</template>

<script>
import { axios } from '@/mixins/axios.js'
import EditorHeader from '@/components/editors/EditorHeader'
import FileUpload from '@/components/editors/FileUpload.vue'
import ImageUpload from '@/components/editors/ImageUpload.vue'
import NoteEditor from '@/components/editors/NoteEditor.vue'
import PollEditor from '@/components/editors/PollEditor.vue'
import SurveyEditor from '@/components/editors/SurveyEditor.vue'

export default {
  name: 'EditorSidebar',
  props: ['boardId', 'editorId'],
  mixins: [axios],
  components: {
    EditorHeader,
    NoteEditor,
    ImageUpload,
    FileUpload,
    PollEditor,
    SurveyEditor
  },
  data () {
    return {
      listener: () => {},
      options: {},
      dueDate: null
    }
  },
  computed: {
    editorType () {
      switch (this.editorId) {
        case 0:
          return this.$t('editor.note.heading')
        case 1:
          return this.$t('editor.image.heading')
        case 2:
          return this.$t('editor.file.heading')
        case 3:
          return this.$t('editor.poll.heading')
        case 4:
          return this.$t('editor.survey.heading')
        default:
          return ''
      }
    }
  },
  methods: {
    // Used to send a note to the backend
    createNote: async function (titleContent, jsonContent) {
      if (titleContent.length < 2) { return }

      var jsonBody
      if (this.dueDate == null) {
        jsonBody = JSON.stringify({
          title: titleContent,
          typeOfPost: 'application/note',
          content: jsonContent
        })
      } else {
        jsonBody = JSON.stringify({
          title: titleContent,
          typeOfPost: 'application/note',
          dueDate: this.dueDate,
          content: jsonContent
        })
      }

      // Post request to api
      await this.axiosPOST('api/boards/' + this.boardId, jsonBody, true, true)
        .then(res => {})
        .catch(err => this.$log.error(err))

      // Notify notice board
      this.$emit('add-note')
    },
    // Used to send an image to the backend
    uploadImage: async function (titleContent, dataURI) {
      if (titleContent.length < 2) { return }

      const dataURISplit = dataURI.split(',')
      const datapart = dataURISplit[0] // E.g. data:image/png;base64
      const base64Data = dataURISplit[1] // BORw0KGgoAAAANSUhEUgAAB...
      const dataType = datapart.split(':')[1].split(';')[0] // Extract MIME type -> image/png

      var jsonBody
      if (this.dueDate == null) {
        jsonBody = JSON.stringify({
          title: titleContent,
          typeOfPost: dataType,
          content: base64Data
        })
      } else {
        jsonBody = JSON.stringify({
          title: titleContent,
          typeOfPost: dataType,
          dueDate: this.dueDate,
          content: base64Data
        })
      }

      // Post request to api
      await this.axiosPOST('api/boards/' + this.boardId, jsonBody, true, true)
        .then(res => {})
        .catch(err => this.$log.error(err))

      // Notify board component
      this.$emit('add-note')
    },
    // Used to send a file to the backend
    uploadFile: async function (titleContent, dataURI) {
      if (titleContent.length < 2) { return }

      const dataURISplit = dataURI.split(',')
      const datapart = dataURISplit[0] // e.g. data:application/pdf;base64
      const base64Data = dataURISplit[1] // ZGgoAAAANSUhEUgAASs54B...
      const dataType = datapart.split(':')[1].split(';')[0] // Extract MIME type -> application/pdf

      var jsonBody
      if (this.dueDate == null) {
        jsonBody = JSON.stringify({
          title: titleContent,
          typeOfPost: dataType,
          content: base64Data
        })
      } else {
        jsonBody = JSON.stringify({
          title: titleContent,
          typeOfPost: dataType,
          dueDate: this.dueDate,
          content: base64Data
        })
      }

      // Post request to api
      await this.axiosPOST('api/boards/' + this.boardId, jsonBody, true, true)
        .then(res => {})
        .catch(err => this.$log.error(err))

      // Notify notice board
      this.$emit('add-note')
    },
    // Used to send a poll draft to the backend
    createPoll: async function (titleContent, jsonContent, answerIndices, answerNames) {
      if (titleContent.length < 2) { return }

      var jsonBodyNote
      if (this.dueDate == null) {
        jsonBodyNote = JSON.stringify({
          title: titleContent,
          typeOfPost: 'application/poll',
          content: jsonContent
        })
      } else {
        jsonBodyNote = JSON.stringify({
          title: titleContent,
          typeOfPost: 'application/poll',
          dueDate: this.dueDate,
          content: jsonContent
        })
      }

      // Post request to api
      await this.axiosPOST('api/boards/' + this.boardId, jsonBodyNote, true, true)
        .then(async postResponse => {
          const jsonBodyPoll = JSON.stringify({
            postId: postResponse.data.id,
            answerIds: answerIndices,
            answers: answerNames
          })
          await this.axiosPOST('api/polls', jsonBodyPoll, true, false)
            .then(pollResponse => {})
            .catch(err => this.$log.error(err))
        })
        .catch(err => this.$log.error(err))

      // Notify notice board
      this.$emit('add-note')
    },
    // Used to send a survey draft to the backend
    createSurvey: async function (titleContent, jsonContent, questionIndices, questions, mcqAnswers) {
      if (titleContent.length < 2) { return }

      const jsonBodyNote = JSON.stringify({
        title: titleContent,
        typeOfPost: 'application/survey',
        content: jsonContent
      })

      // Post request to api
      await this.axiosPOST('api/boards/' + this.boardId, jsonBodyNote, true, true)
        .then(async postResponse => {
          const jsonBodySurvey = JSON.stringify({
            postId: postResponse.data.id,
            questionIds: questionIndices,
            questions: questions,
            answers: mcqAnswers
          })
          await this.axiosPOST('api/survey', jsonBodySurvey, true, false)
            .then(surveyResponse => {})
            .catch(err => this.$log.error(err))
        })
        .catch(err => this.$log.error(err))

      // Notify notice board
      this.$emit('add-note')
    },
    // Called when the date property in the editor header changes
    updateDate: function (date) {
      this.dueDate = (new Date(date)).getTime()
    },
    // This method sends the command to close the editor sidebar to the board component
    close: function () {
      this.$emit('close')
    }
  }
}
</script>

<style scoped>
  hr {
    height: 1px;
    border: none;
    background-color: #aaa;
  }
</style>
