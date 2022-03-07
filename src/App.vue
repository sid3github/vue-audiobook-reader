<template>
    <video
        @playing="videoPlay"
        ref="video"
        width="720"
        height="560"
        autoplay
        muted
    ></video>
    <pre ref="text"></pre>
</template>

<script>
export default {
    methods: {
        async videoPlay() {
            const worker = Tesseract.createWorker();
            await worker.load();
            await worker.loadLanguage("eng");
            await worker.initialize("eng");
            const canvas = document.createElement("canvas");
            canvas.width = this.$refs.video.width;
            canvas.height = this.$refs.video.height;
            document.addEventListener("keypress", async (e) => {
                if (e.code !== "Space") return;
                canvas
                    .getContext("2d")
                    .drawImage(
                        this.$refs.video,
                        0,
                        0,
                        this.$refs.video.width,
                        this.$refs.video.height
                    );
                const {
                    data: { text },
                } = await worker.recognize(canvas);

                speechSynthesis.speak(
                    new SpeechSynthesisUtterance(text.replace(/\s/g, " "))
                );

                this.$refs.text.textContent = text;
            });
        },
    },
    async created() {
        const stream = await navigator.mediaDevices.getUserMedia({
            video: true,
        });
        this.$refs.video.srcObject = stream;
    },
};
</script>

<style>
pre {
    font-size: 2rem;
    font-family: inherit;
    width: 100%;
    white-space: pre-wrap;
}
</style>
