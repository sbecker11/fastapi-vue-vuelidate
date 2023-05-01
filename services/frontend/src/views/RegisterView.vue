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
        <label for="full_name" class="form-label">Full Name:</label>
        <input
          type="text"
          name="full_name"
          v-model="user.full_name"
          class="form-control"
          @focus="v$.user.full_name.$touch()"
        />
        <p v-for="error of v$.user.full_name?.$errors" :key="error.$uid">
          <strong style="color: red;" >{{ error.$message }}</strong>
        </p>
      </div>

      <div class="mb-3">
        <label for="password" class="form-label">Password</label>
        <input
          :type="passwordType"
          v-model="user.password"
          name="password"
          class="form-control"
        />
        <div class="show-box">
          <button @click="showPassword">Show</button>
        </div>
        <label for="strength"
          >Strength<span v-if="!isInitial">: </span>
          <span
            v-bind:class="{
              initial: isInitial,
              short: isShort,
              weak: isWeak,
              fair: isFair,
              excellent: isExcellent,
            }"
            >{{ passwordStregnth }}</span
          >
        </label>
        <div class="rg-bar">
          <div
            v-bind:class="{
              highlight: true,
              initial: isInitial,
              bgShort: isShort,
              bgWeak: isWeak,
              bgFair: isFair,
              bgExcellent: isExcellent,
            }"
          ></div>
        </div>
        <p class="support-text">Your password must:</p>
        <ul class="support-text">
          <li v-bind:class="{ checked: isAtLeast8 }">
            <i v-if="!isAtLeast8">&#10008;</i>
            <i v-else>&#10004;</i>
            Be at least 8 characters long
          </li>
          <li v-bind:class="{ checked: isSpecial }">
            <i v-if="!isSpecial">&#10008;</i>
            <i v-else>&#10004;</i>
            <i class="fas fa-check"></i> Contain a special character
          </li>
          <li v-bind:class="{ checked: isCapital }">
            <i v-if="!isCapital">&#10008;</i>
            <i v-else>&#10004;</i>
            <i class="fas fa-check"></i> Contain a capital letter
          </li>
          <li v-bind:class="{ checked: isExcellent }">
            <i v-if="!isExcellent">&#10008;</i>
            <i v-else>&#10004;</i>
            <i class="fas fa-check"></i> Be excellent
          </li>
        </ul>
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
        :disabled="!isExcellent || v$.$invalid"
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
import { required, sameAs, helpers } from "@vuelidate/validators";

export default defineComponent({
  name: "Register",
  components: {
    PasswordMeter,
  },
  data() {
    return {
      user: {
        username: "",
        full_name: "",
        password: "",
        confirmPassword: "",
      },
      passwordType: "password",
      passwordModelold: "",
      passwordTypeold: "password",
    };
  },
  setup() {
    return { v$: useVuelidate() };
  },
  computed: {
    isInitial() {
      return this.user.password.length < 3;
    },
    isShort() {
      return this.user.password.length >= 3 && !this.isAtLeast8;
    },
    isWeak() {
      return this.isAtLeast8 && (!this.isSpecial || !this.isCapital);
    },
    isFair() {
      return this.isAtLeast8 && this.isSpecial && this.isCapital;
    },
    isExcellent() {
      return (
        this.user.password.length >= 12 && this.isSpecial && this.isCapital
      );
    },
    isValid() {
      return this.isFair || this.isExcellent;
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
    passwordStregnth() {
      let msg = "";
      msg = this.isShort ? "Very Weak" : msg;
      msg = this.isWeak ? "Weak" : msg;
      msg = this.isFair ? "Fair" : msg;
      msg = this.isExcellent ? "Strong" : msg;
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
    showPassword() {
      this.passwordType =
        this.passwordType === "password" ? "text" : "password";
    },
    showPasswordold() {
      this.passwordTypeold =
        this.passwordTypeold === "password" ? "text" : "password";
    },
  },
  validations() {
    return {
      user: {
        username: { required: helpers.withMessage('Username is required', required) }, 
        full_name: { required: helpers.withMessage('Full name is required', required) },
        password: {
          containsPasswordRequirement: helpers.withMessage(
            () =>
              `The password requires at least 8 characters, 1 uppercase, 1 number, 1 special character, and strength must be strong`,
            (value) =>
              /(?=.*[a-z])(?=.*[A-Z])(?=.*[0-9])(?=.*[!@#\$%\^&\*])/.test(value)
          ),
        },
        confirmPassword: {
          // required: helpers.withMessage('Must match password', required), sameAsPassword: sameAs(this.user.password)
          required: helpers.withMessage('Must match password', required), sameAsPassword: helpers.withMessage('Must match password', sameAs(this.user.password))
        }        
      },
    };
  },
});
</script>

<style src="../scss/styles.scss" lang="scss"></style>
