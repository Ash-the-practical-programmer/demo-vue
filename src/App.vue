<script setup>
import AutoComplete from 'primevue/autocomplete';
import { ref, reactive, computed, onMounted, watchEffect } from 'vue'

const STORAGE_KEY = 'notes'
const notes = ref(JSON.parse(localStorage.getItem(STORAGE_KEY) || '[]'))
watchEffect(() => {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(notes.value))
})

const currentNotes = ref()
const currentNote = reactive({
    id: null,
    title: 'Give me a name',
    content: '',
    tags: [],
    fav: false
})

onMounted(() => {
    allNotes()
})

function allNotes() {
    currentNotes.value = notes.value
}

function favNotes() {
    currentNotes.value = notes.value.filter((note) => note.fav === true)
}

const displayDialog = ref(false)
function openDialog() { 
    displayDialog.value = true
}
function closeDialog() {
    displayDialog.value = false
}

const isNewNote = ref(false)
function addNote() {
  isNewNote.value = true
  currentNote.id = Date.now()
  currentNote.title = 'Give me a name'
  currentNote.content = ''
  currentNote.tags = []
  currentNote.fav = false
  openDialog()
}
function editNote(note) {
  currentNote.id = note.id
  currentNote.title = note.title
  currentNote.content = note.content 
  currentNote.tags = note.tags
  currentNote.fav = note.fav
  openDialog()
}

function saveNote() {
  if (isNewNote.value === false) {
    let note = notes.value.find((note) => note.id === currentNote.id)
    let editedNote = Object.assign({}, currentNote)
    notes.value.splice(notes.value.indexOf(note), 1, editedNote)
    currentNotes.value[currentNotes.value.indexOf(note)] = editedNote
  } else {
    let newNote = Object.assign({}, currentNote)
    notes.value.push(newNote)
    isNewNote.value = false
  }
  closeDialog()
}
function removeNote(note) {
  if (currentNotes.value === notes.value) {
    notes.value.splice(notes.value.indexOf(note), 1)
  } else {
    notes.value.splice(notes.value.indexOf(note), 1)
    currentNotes.value.splice(currentNotes.value.indexOf(note), 1)
  }
}

const selectedTag = ref()
const tags = computed(() => {
  let allTags = []
  notes.value.map((note) => allTags = allTags.concat(note.tags))
  let uniqueTags = [...new Set(allTags)]
  return uniqueTags 
})
function filterNotes() {
  currentNotes.value = notes.value.filter((note) => note.tags.includes(selectedTag.value))
}

const foundNotes = ref()
function searchNote(event) {
  setTimeout(() => {
    if (event.query.trim().length) {
      foundNotes.value = notes.value.filter((note) => {
        return note.title.toLowerCase().startsWith(event.query.toLowerCase())
      })
    }
  }, 250)
}
const selectedNote = ref()
function searchNotes() {
  currentNotes.value = [notes.value.find((note)=>note.title === selectedNote.value.title)]
}
</script>

<template>
    <div>
        <Panel header="Notes Writer">
            <Toolbar class="mb-6">
                <template #start>
                    <Button class="mr-3" label="New" icon="pi pi-plus" @click="addNote" />
                    <span class="p-buttonset">
                        <Button class="p-button-success mr-2" label="All notes" icon="pi pi-list" @click="allNotes" />
                        <Button class="p-button-danger" label="Favorites" icon="pi pi-heart" @click="favNotes" />
                    </span>
                </template>
                <template #end>
                    <div class="flex">
    <Dropdown class="mr-3" v-model="selectedTag" :options="tags" placeholder="Filter by tag" @change="filterNotes" @blur="selectedTag = ''" />
    <div class="p-inputgroup flex">
        <span class="p-inputgroup-addon">
            <i class="pi pi-search mr-1 pt-3"></i>
        </span>
        <AutoComplete class="flex-shrink-1" placeholder="Search notes..." field="title" v-model="selectedNote" :suggestions="foundNotes" @complete="searchNote($event)" @item-select="searchNotes" @blur="selectedNote = ''" />
    </div>
  </div>
                </template>
            </Toolbar>
            <div class="flex flex-wrap justify-content-around gap-3">
                <div v-if="!notes.length">No notes have been created yet. Hit the <b>New</b> button to create one.</div>
                <Card class="bg-red-500" v-for="(note, index) in currentNotes" :key="index">
                    <template #title>
                        {{ note.title }}
                    </template>
                    <template #subtitle>
                        <Tag class="mr-2" :value="tag" v-for="tag in note.tags"></Tag>
                    </template>
                    <template #content>
                        <div class="overflow-hidden max-h-5rem" v-html="note.content"></div>
                    </template>
                    <template #footer>
                        <Button class="p-button-rounded p-button-text" v-tooltip.bottom="'Edit'" icon="pi pi-pencil" @click="editNote(note)" />
                        <Button class="p-button-rounded p-button-text p-button-danger" v-tooltip.bottom="'Add to Favorites'" :icon="note.fav ? 'pi pi-heart-fill' : 'pi pi-heart'" @click="note.fav = !note.fav" />
                        <Button class="p-button-rounded p-button-text text-red-500" v-tooltip.bottom="'Delete'" icon="pi pi-trash" @click="removeNote(note)" />
                    </template>
                </Card>
            </div>
        </Panel>
        <Dialog header="Note" v-model:visible="displayDialog" :breakpoints="{'960px': '75vw', '640px': '90vw'}" :style="{width: '50vw'}" :maximizable="true" :modal="true">
  <Inplace :closable="true">
    <template #display>
      <span class="text-xl">{{ currentNote.title }}</span>
    </template>
    <template #content>
      <InputText v-model="currentNote.title" />
    </template>
  </Inplace>
  <Editor class="my-4" v-model="currentNote.content" editorStyle="height: 320px">
    <template #toolbar>
      <span class="ql-formats">
        <button class="ql-bold" v-tooltip.bottom="'Bold'"></button>
        <button class="ql-italic" v-tooltip.bottom="'Italic'"></button>
        <button class="ql-underline" v-tooltip.bottom="'Underline'"></button>
      </span>
    </template>
  </Editor>
  <span class="p-float-label">
    <Chips v-model="currentNote.tags" separator="," class="pr-2"/> 
    <label for="chips">Add tags...</label>
  </span>
  <template #footer>
    <Button class="p-button-text" label="Cancel" icon="pi pi-times" @click="closeDialog" />
    <Button label="Save" icon="pi pi-check" @click="saveNote" />
  </template>
</Dialog>
    </div>
</template>

<style scoped>
.p-button {
   /*  padding: 0.24rem 0.4rem; */
}
</style>
