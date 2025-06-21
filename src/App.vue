<script setup>
import { onMounted, reactive, ref } from "vue";
import { useToast } from "primevue/usetoast";
import { Form } from "@primevue/forms";
import {
  Button,
  InputText,
  Message,
  Toast,
  FileUpload,
  Select,
  DatePicker,
  RadioButton,
  RadioButtonGroup,
} from "primevue";
import { zodResolver } from "@primevue/forms/resolvers/zod";
import { z } from "zod";

const toast = useToast();
const file = ref(null);
const fileError = ref(null);
const form = ref(null);
const isLoading = ref(false);
const occupationLoading = ref(false);
const occupations = ref([]);

const initialValues = reactive({
  firstName: "",
  lastName: "",
  email: "",
  phoneNumber: "",
  sex: null,
  profile: null,
  birthDate: null,
  occupationId: null,
});

const resolver = zodResolver(
  z.object({
    email: z
      .string()
      .min(1, { message: "Please provide a valid Email." })
      .email({ message: "Please provide a valid Email." }),
    phoneNumber: z
      .string()
      .min(10, { message: "Please provide a valid Phone Number." })
      .max(10, { message: "Please provide a valid Phone Number." }),
    birthDate: z.date().refine((date) => date instanceof Date && !isNaN(date), {
      message: "Please provide a valid Birth Date.",
    }),
    occupationId: z.number().refine((value) => value, {
      message: "Please select an Occupation.",
    }),
    firstName: z.string().min(1, { message: "First Name is required." }),
    lastName: z.string().min(1, { message: "Last Name is required." }),
    sex: z.string().refine((value) => value, {
      message: "Please select a Sex.",
    }),
  })
);

const onFileSelect = () => {
  fileError.value = null;
};

const validateFile = (file) => {
  try {
    z.instanceof(File).parse(file);
    return true;
  } catch (err) {
    return false;
  }
};

const onFormSubmit = async (e) => {
  try {
    isLoading.value = true;
    let isValidFile = validateFile(file.value.files[0]);
    if (!isValidFile) {
      fileError.value = {
        invalid: true,
        message: "Please select a valid file.",
      };
      return;
    } else {
      fileError.value = null;
    }
    if (!e.valid) {
      return;
    }
    const formData = new FormData();
    formData.append("profile", file.value.files[0]);
    Object.keys(e.values).forEach((key) => {
      if (key === "birthDate") {
        formData.append(key, e.values[key].toISOString());
      } else {
        formData.append(key, e.values[key]);
      }
    });
    const res = await fetch("http://localhost:5296/User", {
      method: "POST",
      body: formData,
    });
    const json = await res.json();

    if (!json.isSuccess) {
      return toast.add({
        severity: "error",
        summary: json.message,
        detail: json.message,
        life: 3000,
      });
    }

    toast.add({
      severity: "success",
      summary: json.message,
      life: 3000,
    });
    form.value.reset();
    file.value.files = [];
  } catch (err) {
    console.log(err);
  } finally {
    isLoading.value = false;
  }
};

const onResetForm = (e) => {
  e.originalEvent.target.reset();
  file.value.files = [];
  fileError.value = null;
};

const getOccupationList = async () => {
  try {
    occupationLoading.value = true;
    const res = await fetch("http://localhost:5296/occupation/list");
    const json = await res.json();
    if (json.isSuccess) {
      occupations.value = json.data;
    } else {
      toast.add({
        severity: "error",
        summary: json.message,
        detail: json.message,
        life: 3000,
      });
    }
  } catch (err) {
    console.log(err);
  } finally {
    occupationLoading.value = false;
  }
};

onMounted(() => {
  getOccupationList();
});
</script>

<template>
  <div class="card flex flex-col items-center gap-20 justify-center">
    <div
      class="text-2xl font-bold border-4 border-black bg-lime-800 w-full text-center"
    >
      IT 04-1
    </div>
    <Toast />

    <Form
      ref="form"
      v-slot="$form"
      :initialValues
      :resolver
      @submit="onFormSubmit"
      @reset="onResetForm"
      class="flex flex-col gap-4 w-full sm:w-56 md:w-96 lg:w-128"
    >
      <div class="flex gap-5 items-center justify-between">
        <div class="flex flex-col gap-1">
          <label for="firstName">First Name</label>
          <InputText id="firstName" name="firstName" type="text" fluid />
          <Message
            v-if="$form.firstName?.invalid"
            severity="error"
            size="small"
            variant="simple"
            >{{ $form.firstName.error?.message }}</Message
          >
        </div>
        <div class="flex flex-col gap-1">
          <label for="lastName">Last Name</label>
          <InputText id="lastName" name="lastName" type="text" fluid />
          <Message
            v-if="$form.lastName?.invalid"
            severity="error"
            size="small"
            variant="simple"
            >{{ $form.lastName.error?.message }}</Message
          >
        </div>
      </div>

      <div class="flex gap-5 items-center justify-between">
        <div class="flex flex-col gap-1">
          <label for="email">Email</label>
          <InputText id="email" name="email" type="text" fluid />
          <Message
            v-if="$form.email?.invalid"
            severity="error"
            size="small"
            variant="simple"
            >{{ $form.email.error?.message }}</Message
          >
        </div>
        <div class="flex flex-col gap-1">
          <label for="phoneNumber">Phone</label>
          <InputText
            id="phoneNumber"
            name="phoneNumber"
            type="text"
            maxlength="10"
            fluid
          />
          <Message
            v-if="$form.phoneNumber?.invalid"
            severity="error"
            size="small"
            variant="simple"
            >{{ $form.phoneNumber.error?.message }}</Message
          >
        </div>
      </div>

      <div class="flex gap-5 items-center justify-between">
        <div class="flex flex-col gap-1 text-ellipsis">
          <label>Profile</label>
          <FileUpload
            ref="file"
            mode="basic"
            name="profile"
            accept="image/*"
            :auto="false"
            :multiple="false"
            :maxFileSize="1000000"
            @select="onFileSelect"
          />
          <Message
            v-if="fileError?.invalid"
            severity="error"
            size="small"
            variant="simple"
            >{{ fileError?.message }}</Message
          >
        </div>
        <div class="flex flex-col gap-1">
          <label for="birthDate">Birth day</label>
          <DatePicker inputId="birthDate" name="birthDate" fluid />
          <Message
            v-if="$form.birthDate?.invalid"
            severity="error"
            size="small"
            variant="simple"
            >{{ $form.birthDate.error?.message }}</Message
          >
        </div>
      </div>

      <div class="flex gap-5 items-center justify-between">
        <div class="flex flex-col gap-1">
          <label for="occupation">Occupation</label>
          <Select
            name="occupationId"
            filter
            :options="occupations"
            :loading="occupationLoading"
            optionValue="id"
            optionLabel="name"
            class="w-full md:w-56"
          />
          <Message
            v-if="$form.occupationId?.invalid"
            severity="error"
            size="small"
            variant="simple"
            >{{ $form.occupationId.error?.message }}</Message
          >
        </div>
      </div>

      <div class="flex flex-col flex-wrap gap-4">
        <RadioButtonGroup name="sex" class="flex flex-wrap gap-4">
          <div class="flex items-center gap-2">
            <RadioButton inputId="male" value="Male" />
            <label for="male">Male</label>
          </div>
          <div class="flex items-center gap-2">
            <RadioButton inputId="female" value="Female" />
            <label for="female">Female</label>
          </div>
        </RadioButtonGroup>
        <Message
          v-if="$form.sex?.invalid"
          severity="error"
          size="small"
          variant="simple"
          >{{ $form.sex.error?.message }}</Message
        >
      </div>

      <div class="flex gap-4 justify-center">
        <Button
          :loading="isLoading"
          type="submit"
          severity="primary"
          label="Save"
        />
        <Button
          :loading="isLoading"
          type="reset"
          severity="secondary"
          label="Clear"
        />
      </div>
    </Form>
  </div>
</template>
