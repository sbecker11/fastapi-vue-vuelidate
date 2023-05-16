<template>
  <section>
    <form @submit.prevent="submit">
      <div class="mb-3">
        <label for="username" class="form-label">Username:</label>
        <input
          type="text"
          name="username"
          v-model="user.username"
          class="form-control"
          @focus="v$.user.username.$touch()"
        />
        <p v-for="error of v$.user.username?.$errors" :key="error.$uid">
          <strong style="color: red;" >{{ error.$message }}</strong>
        </p>
      </div>

      <div class="mb-3">
        <label for="password" class="form-label" style="width:100%">Password 
          <button style="float:right;font-size:small;border:none" 
            @click="showPassword">{{ this.thePasswordToggleText }}</button>
        </label>
        <input
          :type="passwordType"
          v-model="user.password"
          name="password"
          class="form-control"
          @focus="toggleIsEditingPasswordOn"
          @blur="toggleIsEditingPasswordOff"
        />
        <div class="password-validation" v-if="isEditingPassword">
          <label for="strength" style="margin-top:1em"
            >Strength<span v-if="!isInitial">: </span>
            <span
              v-bind:class="{
                initial: isInitial,
                short: isShort,
                weak: isWeak,
                fair: isFair,
                good: isGood,
                strong: isStrong,
              }"
              >{{ passwordStrength }}</span
            >
          </label>
          <div class="rg-bar">
            <div
              v-bind:class="{
                highlight: true,
                initial: isInitial,
                bgZero: isInitial,
                bgShort: isShort,
                bgWeak: isWeak,
                bgFair: isFair,
                bgGood: isGood,
                bgStrong: isStrong,
              }"
            ></div>
          </div>
          <ul class="support-text" style="margin-top:1em;">
            <li>
              <i v-if="!isAtLeast8" class="error">&#10008;</i>
              <i v-else class="success">&#10004;</i>
              Password must be at least 8 characters long
            </li>
            <li>
              <i v-if="!isSpecial" class="error">&#10008;</i>
              <i v-else class="success">&#10004;</i>
              <i class="fas fa-check"></i> Contain a special character
            </li>
            <li>
              <i v-if="!isCapital" class="error">&#10008;</i>
              <i v-else class="success">&#10004;</i>
              <i class="fas fa-check"></i> Contain a capital letter
            </li>
            <li>
              <i v-if="!isDigit" class="error">&#10008;</i>
              <i v-else class="success">&#10004;</i>
              <i class="fas fa-check"></i> Contain a digit
            </li>
            <li>
              <i v-if="!isStrong" class="error">&#10008;</i>
              <i v-else class="success">&#10004;</i>
              <i class="fas fa-check"></i> Be Strong
            </li>
          </ul>
        </div>
      </div>

      <div class="mb-3">
        <label for="confirmPassword" class="form-label">Confirm Password</label>
        <input
          :type="passwordType"
          v-model="user.confirmPassword"
          name="confirmPassword"
          class="form-control"
          @focus="v$.user.confirmPassword.$touch()"
        />
        <p v-for="error of v$.user.confirmPassword?.$errors?.slice(0, 1)" :key="error.$uid">
          <strong style="color: red;" >{{ error.$message }}</strong>
        </p>        
      </div>

      <button
        type="submit"
        :disabled="!isStrong || v$.$invalid"
        class="btn btn-primary"
      >
        Submit
      </button>
    </form>
  </section>
</template>

<script>
import { defineComponent, ref } from "vue";
import { mapActions } from "vuex";
import PasswordMeter from "vue-simple-password-meter";
import { useVuelidate } from "@vuelidate/core";
import { required, minLength, sameAs, helpers } from "@vuelidate/validators";

export default defineComponent({
  name: "Register",
  components: {
    PasswordMeter,
  },
  data() {
    return {
      user: {
        username: "",
        password: "",
        confirmPassword: "",
      },
      passwordType: "password",
      passwordModelold: "",
      passwordTypeold: "password",
      thePasswordToggleText: "Show",
      isEditingPassword: "",
    };
  },
  setup() {
    return { v$: useVuelidate() };
  },
  computed: {
    isInitial() {
      return this.user.password.length == 0;
    },
    isShort() {
      return this.user.password.length >= 1;
    },
    isWeak() {
      // any one match
      return  (this.isSpecial || this.isCapital || this.isDigit);
    },
    isFair() {
      // any two match
      if (this.isSpecial && this.isCapital) return true;
      if (this.isSpecial && this.isDigit) return true;
      if (this.isCapital && this.isDigit) return true;
      return false;
    },
    isGood() {
      // all 3 match
      return this.isSpecial && this.isCapital && this.isDigit;
    },
    isStrong() {
     return this.user.password.length >= 8 && this.isSpecial && this.isCapital  && this.isDigit;
    },
    isValid() {
      return this.isFair || this.isStrong;
    },
    isDigit() {
      const regex = /[0-9]/g;
      return this.user.password.match(regex);
    },
    isCapital() {
      const regex = /[A-Z]/g;
      return this.user.password.match(regex);
    },
    isAtLeast8() {
      return this.user.password.length >= 8;
    },
    isSpecial() {
      const regex = /[^A-Za-z0-9]/g;
      return this.user.password.match(regex);
    },
    passwordStrength() {
      let msg = "";
      msg = this.isShort ? "Very Weak" : msg;
      msg = this.isWeak ? "Weak" : msg;
      msg = this.isFair ? "Fair" : msg;
      msg = this.isGood ? "Good" : msg;
      msg = this.isStrong ? "Strong" : msg;
      return msg;
    },
  },
  methods: {
    ...mapActions(["register"]),
    async submit(e) {
      e.preventDefault();
      const validForm = await this.v$.$validate();
      if (validForm) {
        try {
          await this.register(this.user);
          this.$router.push("/dashboard");
        } catch (error) {
          throw "Username already exists. Please try again.";
        }
      }
    },
    updatePasswordToggleText() {
      this.thePasswordToggleText =
        this.passwordType === "password" ? "Show" : "Hide";
    },
    showPassword() {
      this.passwordType =
        this.passwordType === "password" ? "text" : "password";
      this.updatePasswordToggleText();
    },
    showPasswordold() {
      this.passwordTypeold =
        this.passwordTypeold === "password" ? "text" : "password";
      this.updatePasswordToggleText();
    },
    toggleIsEditingPasswordOn() {
      this.isEditingPassword = true;
      console.log("isEditingPasswordOn: " + this.isEditingPassword)
    },
    toggleIsEditingPasswordOff() {
      console.log("isStrong:" + this.isStrong);
      if ( this.isStrong ) {
        this.isEditingPassword = false;
        console.log("isEditingPasswordOff: " + this.isEditingPassword)
      }
    }
  },
  validations() {
    return {
      user: {
        username: { 
          required: helpers.withMessage('Username is required', required), 
          minLength: minLength(3),
          containsNoSpaces: helpers.withMessage(
            () =>
              `Username can have no spaces`,
            (value) =>
              /^(|\S+)$/.test(value)
            // if value is not blank then is must contain no spaces
            // value is valie if it is blank or it has no spaces
          ),
        }, 
        password: {
          containsPasswordRequirement: helpers.withMessage(
            () =>
              `The password requires at least 8 characters, 1 uppercase, 1 digit, 1 special character, and strength must be strong`,
            (value) =>
              /(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#\$%\^&\*])/.test(value)
          ),
        },
        confirmPassword: {
          required: helpers.withMessage('Must match password', required), 
          sameAsPassword: helpers.withMessage('Must match password', sameAs(this.user.password))
        }        
      },
    };
  },
});
</script>

<style src="../scss/styles.scss" lang="scss"></style>
