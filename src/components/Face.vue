<template>
            <v-layout row wrap>
                    <v-flex xs12 sm12 md6 lg6 xl6>
                         <v-card>
                        <video id="video" width="100%" height="50%" autoplay></video>
                        <canvas id="canvas" width="600" height="480" style="display: none;"></canvas>
                         </v-card>
                        <v-btn @click.native="process" block secondary dark>
                            <v-icon left>camera_alt</v-icon> Analyze</v-btn>
                    </v-flex>
                    <v-flex xs12 sm12 md6 lg6 xl6>
                        <h2 class="orange--text text-xs-center">Result</h2>
                        <div class="text-xs-center" v-show="loader">
                            <v-progress-circular indeterminate v-bind:size="100" v-bind:width="3" class="teal--text"></v-progress-circular>
                        </div>
                        <div v-show="result" class="text-md-center">
                            <p>Phẫn nộ: <span class="blue--text">{{anger}}</span></p>
                            <p>Đéo rõ: <span class="blue--text"> {{blur}}</span></p>
                            <p>Vui sướng: <span class="blue--text">  {{joy}} </span> </p>
                            <p>Đau buồn: <span class="blue--text">  {{sorrow}} </span> </p>
                            <p>Ngạc nhiên: <span class="blue--text">  {{surprised}} </span> </p>
                            <h4 class="red--text">Confidence: {{ confidence }} %</h4>
                        </div>
                    </v-flex>
                </v-layout>
</template>

<script>
import $ from "jquery";
import axios from 'axios';
export default {
  data(){
      return{
                loader: false,
                result: false,
                apiKey: "",  //google cloud api  Browser key
                anger: null,
                blur: null,
                headwear: null,
                sorrow: null,
                joy: null,
                surprised: null,
                confidence: null,
                data: {               //type vision api Request
                    "requests": [{
                        "features": [{
                            "type": "FACE_DETECTION"
                        }],
                        "image": {
                            "content": null
                        }
                    }]
                }
      }
  },
              methods: {
                process() {
                    const canvas = document.getElementById('canvas');
                    const context = canvas.getContext('2d');
                    const vm = this;
                    this.result = false;
                    this.loader = true;
                    context.drawImage(video, 0, 0, 640, 480);
                    const base64 = canvas.toDataURL();
                    const finalImage = base64.replace("data:image/png;base64,", "");
                    this.data.requests[0].image.content = finalImage;
                    axios.post(`https://cxl-services.appspot.com/proxy?url=${encodeURIComponent('https://vision.googleapis.com/v1/images:annotate')}`,
                        this.data).then(response => {
                        const result = response.data.responses[0].faceAnnotations[0];
                        vm.anger = result.angerLikelihood;
                        vm.blur = result.blurredLikelihood;
                        vm.headwear = result.headwearLikelihood;
                        vm.joy = result.joyLikelihood;
                        vm.sorrow = result.sorrowLikelihood;
                        vm.surprised = result.surpriseLikelihood;
                        vm.confidence = this.confidenceInt(result.detectionConfidence);
                        vm.loader = false;
                        vm.result = true;
                        console.log(result);
                    }).catch(error => {
                        console.log(error);
                    })
                },
                confidenceInt(num){
                   const dig = num.toFixed(2);
                   if(dig == 1.0){
                    return 100;
                   }else{
                   const str = dig.toString();
                   return str.substring(2, 4);
                   }

                }
            }
}
 $(function() {
        const video = document.getElementById('video');
        if (navigator.mediaDevices && navigator.mediaDevices.getUserMedia) {
           navigator.mediaDevices.getUserMedia({
               video: true
           }).then(stream => {
               video.src = window.URL.createObjectURL(stream);
                video.play();
            });
        }

     });

</script>


<style>
p {
    font-size: 18px
}
</style>
