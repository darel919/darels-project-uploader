<template>
    <div class="store-parent p-12">
        <URadioGroup v-model="mode" :options="methods" class="mb-4">
            <template #label="{ option }">
                <section class="flex items-center">
                    <UIcon :name="option.icon" class="w-6 h-6" />
                    <p class="ml-2 text-md">
                    {{ option.label }}
                </p>
                </section>
            </template>
        </URadioGroup>
        <section v-if="mode === 'selfhost'">
            <h1 class="font-bold text-2xl">New Video Upload Wizard</h1>
            <h2>Upload videos to darel's Video Project server</h2>
            <UForm  class="store-content my-4"  :state="state" @submit="goUpload()">
                <UInput type="file" size="sm" icon="i-heroicons-folder" id="fileInput" class="py-4"/>
                <UFormGroup name="title" class="store-item p-4 my-2" label="Content title">
                    <UInput type="text" placeholder="Ulang tahun Adib ke-60" v-model="state.title" required/>
                </UFormGroup>
                <UFormGroup name="desc" class="store-item p-4 my-2" label="Content description">
                    <UInput type="text" placeholder="Recorded at 32th April, 2062" v-model="state.desc" required/>
                </UFormGroup>
                <UFormGroup name="date_created" class="store-item p-4 my-2" label="Date created" description="When is it created?">
                    <UInput type="date" v-model="state.date_created" required/>
                </UFormGroup>
                <UFormGroup name="category_select" class="store-item p-4 my-2" label="Category" description="What is the category?">
                    <USelect v-model="state.category_select" :options="categories" option-attribute="name"/>
                </UFormGroup>
                <UFormGroup name="submitted_by" class="store-item p-4 my-2" label="Content made by" description="Who created it?">
                    <UInput type="text" placeholder="Who makes the content?" v-model="state.submitted_by" required/>
                </UFormGroup>
                <UButton variant="outline" type="submit" :disabled="clicked" :loading="uploading">
                    <p>Upload</p>
                </UButton>
            </UForm>
        </section>
        <section v-if="mode === 'youtube'">
            <h1 class="font-bold text-2xl">New Video Store Wizard</h1>
            <h2>Store YouTube videos data to darel's Video Project server</h2>
            <UForm class="store-content my-4" :state="state" @submit="goStore()">
                <UFormGroup name="ytLink" class="store-item p-4 my-2" label="YouTube Video ID" description="The link of the YouTube video.">
                    <UInput type="text" placeholder="https://youtu.be/TMd7_W_q4Os" v-model="state.ytLink" required autofocus/>
                </UFormGroup>
                <UFormGroup name="date_created" class="store-item p-4 my-2" label="Date created" description="When is it created?">
                    <UInput type="date" v-model="state.date_created" required/>
                </UFormGroup>
                <UFormGroup name="category_select" class="store-item p-4 my-2" label="Category" description="What is the category?">
                    <USelect v-model="state.category_select" :options="categories" option-attribute="name"/>
                </UFormGroup>
                <UFormGroup name="submitted_by" class="store-item p-4 my-2" label="Content made by" description="Who created it?">
                    <UInput type="text" placeholder="Who makes the content?" v-model="state.submitted_by" required/>
                </UFormGroup>
                
                <UButton variant="outline" type="submit">
                    <p>Upload</p>
                </UButton>
            </UForm>
        </section>
    </div>
</template>

<script setup>

const methods = ref([
  { value: 'selfhost', label: 'Self Host', icon:'i-heroicons:server-stack' },
  { value: 'youtube', label: 'YouTube', icon:'i-heroicons:play-solid' },
])
const isHome = ref(true)
// const APIEndpoint = "http://10.10.10.10:328"
function homeDetection() {
    if(isHome.value) {
        methods.value = [
            { value: 'selfhost', label: 'Self Host', icon:'i-heroicons:server-stack' },
            { value: 'youtube', label: 'YouTube', icon:'i-heroicons:play-solid' },
        ]
    } else {
        methods.value = [
            { value: 'youtube', label: 'YouTube', icon:'i-heroicons:play-solid' },
        ]
        mode.value = 'youtube'
    }
}
const mode = ref('selfhost')
const content_id = ref(null);
const categories = [{
    name: "Archives",
    value: "ae43bd4d-0e27-458b-a2ba-6578db189e4f"
}]
const state = reactive({
    title: '',
    desc: '',
    ytLink: null,
    date_created: '2020-01-01',
    category_select: 'ae43bd4d-0e27-458b-a2ba-6578db189e4f',
    submitted_by: 'darelisme Archives',
})
const clicked = ref(false);
const uploading = ref(false);
async function goUpload() {
    try {
        clicked.value = true;
        uploading.value = true;
        const initialData = {
            date_created: state.date_created,
            category: state.category_select,
            submitted_by: state.submitted_by,
            desc: state.desc,
            title: state.title,
            isSelfHost: true
        }
        await $fetch("/dp/store", {
            method: "POST",
            body: initialData,
            baseURL: useRuntimeConfig().public.APIEndpoint,
            async onResponse({request, response, options}) {
                if(response.status == 200) {
                    content_id.value = response._data.upload_id
                    const fileInput = document.getElementById('fileInput');
                    const file = fileInput.files[0];
                    const formData = new FormData();
                    formData.append('file', file);
                    formData.append('content_id', content_id.value)

                    await $fetch("/dp/upload", {
                        method: 'POST',
                        body: formData,
                        baseURL: useRuntimeConfig().public.APIEndpoint,
                        async onRequestError({request, options,error}) {
                            uploading.value = false;
                            clicked.value = false;
                            console.error(error)
                            alert(error)
                        },
                        async onResponse({request, response, options}) {
                            uploading.value = false;
                            clicked.value = false;
                            if(response.status == 200) {
                                console.log(response._data)
                                alert(response._data.message)
                            }
                        },
                        async onResponseError({request, response, options}) {
                            uploading.value = false;
                            clicked.value = false;
                            console.error(response)
                            alert(response.statusText)
                        }
                    })
                }
            },
        })
    } catch (e) {
        clicked.value = false;
        uploading.value = false;
        alert(e)
        console.error(e)
    }
}
async function goStore() {
    if(state.ytLink && state.date_created && state.submitted_by) {
        clicked.value = true;
        const submissionData = {
            type_id: state.category_select,
            v: state.ytLink,
            date: state.date_created,
            submittedBy: state.submitted_by,
            isSelfHost: false
        }
        await $fetch("/dp/store", {
            method: 'POST',
            body: JSON.stringify(submissionData),
            baseURL: useRuntimeConfig().public.APIEndpoint,
            async onRequestError({request, options,error}) {
                console.error(error)
                alert(error)
            },
            async onResponse({request, response, options}) {
                if(response.status == 200) {
                    console.warn("Store success!")
                    // console.log(response)
                    navigateTo('https://projects.darelisme.my.id/watch?v='+response._data.upload_id, {
                        external: true,
                        open: {
                            target: "_blank"
                        }
                    })
                } else {
                    alert(response.statusText)
                }
            },
            async onResponseError({request, response, options}) {
                console.error(response)
                // alert(response.body)
                alert(response.statusText)
            }
        })
    } else {
        alert("Incorrect format!\nCheck form before uploading to server.")
    }
}
onMounted(() => {
homeDetection()
    const head = {
        title: 'Video Upload Wizard'
    }
    useHead(head)
})
</script>

<style scoped>
#disabled {
    opacity: 0.5;
    pointer-events: none;
    user-select: none;
}
.store-content {
    display: flex;
    flex-direction: column;
    /* max-width: 720px; */
}
.store-item {
    border-radius: 10px;
}
.store-item input, select {
    width: 50%;
}
select{
    padding: 10px;
    border-radius:50px;
    background-color: inherit;
    color:var(--color-text);
    border:1px solid var(--color-background-mute)
}
</style>