<template>
    <main class="pb-20">
        <section
            class="flex flex-col bg-purple rounded-main w-3/5 mx-auto p-8 pb-0 max-md:w-4/5 max-md:p-4 max-md:pb-8"
        >
            <h1
                class="font-main font-semibold text-white text-5xl text-center mb-12 max-md:mb-8"
            >
                Créer un meme
            </h1>
            <div class="flex relative">
                <form
                    @submit.prevent="addMeme()"
                    class="flex flex-col w-2/3 gap-8 px-12 pt-4 pb-6 max-md:w-full max-md:px-8 max-md:pt-2"
                >
                    <MyInput
                        name="name"
                        placeholder="Nom"
                        v-model="meme.name"
                    />
                    <FileInput name="image" @file-selected="meme.image = $event" />
                    <MyInput
                        name="text-top"
                        placeholder="Texte du haut"
                        v-model="meme.textTop"
                    />
                    <MyInput
                        name="text-bottom"
                        placeholder="Texte du bas"
                        v-model="meme.textBottom"
                    />
                    <MySelect
                        placeholder="Tags"
                        :tags="tags"
                        @updated="(value) => (meme.tags = value)"
                    />
                    <SubmitButton name="Créer" />
                </form>
                <img
                    class="w-1/3 max-md:hidden self-end"
                    src="/public/illustration1.svg"
                    alt="illustration"
                />
            </div>
        </section>
    </main>
</template>

<script>
import MySelect from "../components/form/MySelect.vue";
import FileInput from "../components/form/FileInput.vue";
import MyInput from "../components/form/MyInput.vue";
import SubmitButton from "../components/form/SubmitButton.vue";
import { myFetch } from "../composable/http";
export default {
    components: {
        MySelect,
        FileInput,
        MyInput,
        SubmitButton,
    },

    data() {
        return {
            meme: {
                name: "",
                image: "",
                textTop: "",
                textBottom: "",
                tags: [],
            },
            tags: [],
            tagsId: [],
        };
    },

    methods: {
        async addMeme() {
            const data = new FormData();
            console.log(data);
            console.log(this.meme.image + 'image');
            // Récupérer les identifiants des tags sélectionnés
            this.tagsId = this.meme.tags.map(({ id }) => id);
            const tagsArray = this.tagsId.map(String);
            console.log(tagsArray);
            data.append("name", this.meme.name);
            data.append("image", this.meme.image);
            data.append("topText", this.meme.textTop);
            data.append("bottomText", this.meme.textBottom);
            data.append("tags", JSON.stringify(tagsArray));
            if (
                data.get("name") === "" ||
                data.get("image") === null ||
                data.get("textTop") === "" ||
                data.get("textBottom") === "" ||
                data.get("tags") === ""
            ) {
                alert("Veuillez remplir tous les champs");
                return;
            }
            console.log(data);

            myFetch("http://localhost:3000/memes/create", {
                method: "POST",
                headers: {
                    "Content-Type": "multipart/form-data",
                },
                body: data,
            })
                .then((response) => {
                    if (!response.ok) {
                        throw new Error("Erreur lors de la création du meme");
                    }
                    // Réinitialisez le formulaire
                    this.meme.name = "";
                    this.meme.image = "";
                    this.meme.textTop = "";
                    this.meme.textBottom = "";
                    this.meme.tags = [];
                })
                .catch((error) => {
                    console.error(error);
                    // Affichez un message d'erreur à l'utilisateur
                    alert("Erreur lors de la création du meme");
                });
                alert("meme crée avec succès");
        },
        async getTags() {
            const response = await myFetch("http://localhost:3000/tags");
            const tags = await response.json();
            this.tags = tags;
        },
    },

    created() {
        this.getTags();
    },
};
</script>
