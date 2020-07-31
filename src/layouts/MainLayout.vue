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
     <q-btn @click="takePic()" size="75px" color="blue-grey-3" class="justify-center" >
        <q-icon name="photo_camera" size="130px"/>
        <br>
      <div class="text-weight-thin text-caption">Pulse para capturar imagen</div>
     </q-btn>
    </q-page-container>
    
    <q-dialog v-model="expanded"  >
      <q-card style="width: 100vw">
        <q-card-section class="row items-center q-pb-none">
          <div class="text-h7 text-primary">Enviar por correo</div>
          <q-space />
          <q-btn flat icon="close" color="black" @click="expanded = false"/>
        </q-card-section>
         <q-card-section>
           <div>
             <q-btn @click="sendMail()"> ENVIAR</q-btn>
          <q-input type="email" label="De" v-model="datosImagen.from"></q-input>
          <q-input type="email" label="Para" v-model="datosImagen.to">></q-input>
          <q-input label="Asunto" v-model="datosImagen.subject">></q-input>
          <q-input text-area label="Texto" v-model="datosImagen.body"> ></q-input>
          </div>
           <q-space />
        </q-card-section>
        <q-card-section size="50px" >
          <q-img :src="datosImagen.data" />
        </q-card-section>
        <q-card-section size="50px" > 
          <div class="justify-center text-center">
            <q-btn @click="sendMail()" icon-right="send" label="ENVIAR" color="primary"/>
          </div>
        </q-card-section>
      </q-card>
    </q-dialog>
  </q-layout>
</template>

<script>
import { date } from 'quasar'
export default {
  data(){ 
    return { 
      options: {},
      expanded: false,
      datosImagen: { 
        from: 'jvilata@edicom.es',
        to: 'jvilata@edicom.es',
        subject: 'subject prueba',
        body: 'texto prueba',
        data: '',
        oMyBlob: null
      }
    }
  },
  methods: { 
    takePic() { 
      this.options.destinationType = Camera.DestinationType.DATA_URL
      navigator.camera.getPicture(
            (data) => { // on success    
              var contentType = 'image/jpeg'          
              this.datosImagen.data = 'data:' + contentType + ';base64,' + data
              this.expanded = true

              
              const img = new Image()
              img.src = this.datosImagen.data
              img.onload = () => {
                const scaleFactor = 1
                // ( img.width > 600 ? 600.0 / img.width : 1)
                const width = img.width * scaleFactor
                const height = img.height * scaleFactor
                const elem = document.createElement('canvas')
                elem.width = width
                elem.height = height
                const ctx = elem.getContext('2d')
                // img.width and img.height will contain the original dimensions
                ctx.drawImage(img, 0, 0, width, height)
                var data = ctx.canvas.toDataURL(contentType, 0.1)
                data = data.substring(('data:'+contentType+';base64,').length)
                console.log(data)
                var raw = atob(data)
                var rawLength = raw.length;
                var uInt8Array = new Uint8Array(rawLength)
                for (var i = 0; i < rawLength; ++i) {
                    uInt8Array[i] = raw.charCodeAt(i)
                }
                this.datosImagen.oMyBlob = new Blob([uInt8Array], {type: contentType})
              }
            },
            () => { // on fail
            },
            this.options)
    },
    sendMail(){
      const headerFormDataSinCredentials = {
        withCredentials: true,
        headers: {
          'Content-Type': 'multipart/form-data'
        }
      }
      var formData = new FormData()
      formData.append('action', 'sendattach')
      formData.append('from', this.datosImagen.from)
      formData.append('to', this.datosImagen.to)
      formData.append('subject', this.datosImagen.subject)
      formData.append('body', this.datosImagen.body)
      formData.append('attach', this.datosImagen.oMyBlob, `ticket ${date.formatDate(new Date(),'YYYY-MM-DD HH:mm:ss')}.jpeg`)
      return this.$axios.post('lib/sendmail.php', formData, headerFormDataSinCredentials)
        .then(response => {
          if (response.data.success) {
            this.$q.dialog({ title: 'OK', message: 'Ticket subido correctamente' })
          } else {
            this.$q.dialog({ title: 'Error', message: 'Error al subir ticket. Vuélvalo a intentar o contacte con el administrador' })
          }
        })
        .catch(error => {
          this.$q.dialog({ title: 'Error', message: 'Error al subir ticket. Vuélvalo a intentar o contacte con el administrador' })
        })   
    }
  }
}
</script>
