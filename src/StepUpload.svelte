<script lang="ts">
    import { ImageStatus } from "./types.d";
    import { imageStatus, modifiedImage, originalImage } from "./store";
    import Dropzone from "dropzone";
    import "dropzone/dist/dropzone.css";
    import { Cloudinary } from "@cloudinary/url-gen";
    import { backgroundRemoval } from "@cloudinary/url-gen/actions/effect";

    import { onMount } from "svelte";

    const cloudinary = new Cloudinary({
        cloud: {
            cloudName: "selecdev",
        },
        url: {
            secure: true,
        },
    });

    onMount(() => {
        const dropzone = new Dropzone("#dropzone", {
            uploadMultiple: false,
            acceptedFiles: ".jpeg,.jpg,.png,.webp",
            maxFiles: 1,
        });

        dropzone.on("sending", (file, xhr, formData) => {
            // aqui podemos añadir la apiKey, configuracion, etc
            formData.append("upload_preset", "selecdefault");
            formData.append("timespam", Date.now() / 1000);
            formData.append("api_key", 192496598861922);
            imageStatus.set(ImageStatus.UPLOADING);
        });

        dropzone.on("success", (file, response) => {
            console.log(response);
            const { public_id: publicId, secure_url: url } = response;

            originalImage.set(url);

            // Crear la imagen con fondo transparente
            const imageWithoutBackground = cloudinary
                .image(publicId)
                .effect(backgroundRemoval());

            // Save and print the modified image.
            imageWithoutBackground.toURL();
            modifiedImage.set(imageWithoutBackground.toURL());

            // Change status to done
            imageStatus.set(ImageStatus.DONE);
        });

        dropzone.on("error", (file, response) => {
            console.log("Ha ido mal");
            console.log(response);
        });
    });
</script>

<form
    action="https://api.cloudinary.com/v1_1/selecdev/image/upload"
    id="dropzone"
    class="shadow-2xl border-dashed border-2 border-gray-300 rounded-lg aspect-video w-full flex items-center justify-center flex-col"
>
    {#if $imageStatus == ImageStatus.READY}
        <button
            class="font-bold pointer-events-none bg-blue-600 rounded-full text-bold text-white text-xl px-6 py-4"
            >Upload files</button
        >
        <strong>or drop a file</strong>
    {/if}
    {#if $imageStatus == ImageStatus.UPLOADING}
        <strong class="text-lg mt-4 text-gray-800">Uploading files...</strong>
    {/if}
</form>
