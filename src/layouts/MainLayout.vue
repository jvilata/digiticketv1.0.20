<template>
  <q-layout view="lHh Lpr lFf">
    <q-header elevated class="bg-color-primary">
      <q-toolbar>
        <q-toolbar-title class="text-center"> 
          DigiTiket 
        </q-toolbar-title>
        </q-toolbar>
    </q-header>
    <q-page-container class="absolute-center">
     <q-btn @click="takePic()" round size="70px" color="blue-grey-5" >
       <q-icon name="photo_camera" size="130px"  />
     </q-btn>
    </q-page-container>
    
    <q-dialog v-model="expanded"  >
      <q-card style="width: 100vw">
        <q-card-section class="row items-center q-pb-none">
          <div class="text-h7 text-primary">Enviar por correo</div>
          <q-space />
          <q-btn flat icon="close" color="primary" @click="expanded = false"/>
        </q-card-section>
         <q-card-section>
           <div>
          <q-input type="email" label="De"></q-input>
          <q-input type="email" label="Para"></q-input>
          <q-input label="Asunto"></q-input>
          <q-input text-area ></q-input>
          </div>
           <q-space />
        </q-card-section>
        <q-card-section size="50px" >
          <q-img :src="datosImagen.data" />
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-layout>
</template>

<script>
export default {
  data(){ 
    return { 
      options: {},
      expanded: false,
      datosImagen: { 
        data: ''
      }
    }
  },
  methods: { 
    takePic() { 
      this.options.destinationType = Camera.DestinationType.DATA_URL
      navigator.camera.getPicture(
            (data) => { // on success              
              this.datosImagen.data = 'data:image/jpeg;base64,' + data
              this.expanded = true
              //window.location.href='mailto:marta.vilata@hotmail.com?subject=Informacion JPersonal App'

            },
            () => { // on fail
              
            },
            this.options)
    }
  }
}
</script>
