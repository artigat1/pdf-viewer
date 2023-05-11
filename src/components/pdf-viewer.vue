<script>
import {
    defineComponent,
    toRaw
} from 'vue'

import {
    EventBus,
    NullL10n,
    PDFFindController,
    PDFLinkService,
    PDFViewer
} from 'pdfjs-dist/web/pdf_viewer'
import {
    getDocument,
    GlobalWorkerOptions
} from 'pdfjs-dist'

const CMAP_URL = '../../../node_modules/pdfjs-dist/cmaps'
GlobalWorkerOptions.workerSrc = `//cdnjs.cloudflare.com/ajax/libs/pdf.js/3.6.172/pdf.worker.js`

export default defineComponent({
    name: 'pdf-viewer',

    data: () => ({
        documentUrl: '/data/test.pdf',
        pdfLoadingTask: null,
        pdfDocument: null,
        pdfViewer: null,
        pdfHistory: null,
        findController: null,
        pdfLinkService: null,
        eventBus: null,
        pageNumber: 0,
        totalPages: 0,

    }),

    async mounted() {
        await this.loadPdf()
    },

    methods: {
        async loadPdf() {
            const pdfLoadingTask = getDocument({
                url: this.documentUrl,
                cMapUrl: CMAP_URL,
                cMapPacked: true,
                maxImageSize: Number.MAX_VALUE,
                disableAutoFetch: true,
                disableStream: true,
            })

            const pdfDocument = await pdfLoadingTask.promise

            const container = this.$refs.container
            const eventBus = new EventBus()
            const linkService = new PDFLinkService({
                eventBus,
            })
            const findController = new PDFFindController({
                eventBus,
                linkService,
            })
            const pdfViewer = new PDFViewer({
                container,
                eventBus,
                linkService,
                findController,
            })

            eventBus.on('pagesinit', () => {
                pdfViewer.currentScaleValue = 'page-fit'
                const initialZoomValue = pdfViewer.currentScale * 100
                const currentZoomLevel = (Math.round(initialZoomValue / 10) * 10)
                console.log('PDF Initialised', initialZoomValue, currentZoomLevel)
            })

            pdfViewer.setDocument(pdfDocument)

            this.totalPages = pdfDocument.numPages
            this.pageNumber = pdfViewer.currentPageNumber
        },
    },
})
</script>

<template>
    <section>
        <p class="page-numbers">{{ pageNumber }} / {{ totalPages }}</p>
        <div ref="container"
             class="pdf-viewer-container">
            <div id="viewer" class="pdf-viewer"></div>
        </div>
    </section>
</template>

<style scoped>
body {
    background-color: #808080;
    margin: 0;
    padding: 0;
}

.page-numbers{
    position: absolute;
    top: 0;
    left: 16px;
    height: 30px;
}

.pdf-viewer-container {
    overflow: auto;
    position: absolute;
    left: 16px;
    top: 50px;
    width: 100vw;
    height: calc(100vh - 50px);
}
</style>
