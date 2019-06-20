<template>
    <div>
        <Upload
          multiple
            ref="upload"
            :before-upload="handleUpload"
            :on-remove="handleRemove"
            action="//jsonplaceholder.typicode.com/posts/">
            <Button icon="ios-cloud-upload-outline">Select the file to upload</Button>
        </Upload>
        <div v-if="file !== null">Upload file: {{ file.name }} <Button type="text" @click="upload" :loading="loadingStatus">{{ loadingStatus ? 'Uploading' : 'Click to upload' }}</Button></div>
        <div v-if="files !== []">Upload file(s): 
          <div v-for="(file, index) in files" :key="index">
            {{ file.name }} 
          </div>
          <Button type="text" @click="upload" :loading="loadingStatus">{{ loadingStatus ? 'Uploading' : 'Click to upload' }}</Button>
        </div>
        <Button icon="ios-trash-outline" @click="clearList">Clear Upload List</Button>
    </div>
</template>
<script>
import { createReadStream, createWriteStream } from 'fs'
import sha256 from 'simple-sha256' 
import * as path from 'path'
import readline from 'readline'
import { bindCallback } from 'rxjs'
    export default {
        data () {
            return {
                file: null,
                files:[],
                loadingStatus: false
            }
        },
        methods: {
            handleUpload (file) {
               this.files.push(file)
                return false;
            },
            handleRemove(file, fileList) {
              console.log(file, fileList)
            },
            clearList () {
              this.files = []
              this.$nextTick(() => { this.uploadList = this.$refs.upload.fileList; })
            },
            upload () {
                // this.loadingStatus = true;
                const reader = createReadStream(this.files[0].path)
                const { dir, ext, name} = path.parse(this.files[0].path)
                const hashedFile = path.join(dir, `${name}-HASHED${ext}`)
                const writer = createWriteStream(hashedFile, { encoding: 'utf8', flags: 'a'})
          
                const rl = readline.createInterface({
                  input: reader,
                  output: writer,
                  terminal: false
                })

                rl.on('line', async (line) => {
                  
                  let email = line.split(",")[0]
                  if(email === '' || !email.includes('@')) {

                  } else {
                    let hash = await sha256(email)
                    writer.write(hash)
                  }
                })
          //  reader.on('data', (chuck) => {
              //    console.log("Chunk", chuck.toString())
              //  })
                // setTimeout(() => {
                //     this.file = null;
                //     this.loadingStatus = false;
                //     this.$Message.success('Success')
                // }, 1500);
            }
        }
    }
</script>