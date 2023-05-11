<script>
import { defineComponent } from 'vue'

import * as pdfjsLib from 'pdfjs-dist/build/pdf.js'
import * as pdfjsViewer from 'pdfjs-dist/web/pdf_viewer'

const CMAP_URL = '../../../../node_modules/pdfjs-dist/cmaps'
pdfjsLib.GlobalWorkerOptions.workerSrc = `//cdnjs.cloudflare.com/ajax/libs/pdf.js/3.6.172/pdf.worker.js`

export default defineComponent({
    name: 'pdf-viewer',

    data: () => ({
        token: 'Bearer eyJhbGciOiJSUzI1NiIsImtpZCI6Inp6NGZkakd5ZHdJTm9QSGppU1hMeUEiLCJ0eXAiOiJhdCtqd3QifQ.eyJuYmYiOjE2ODM4MTY1MTUsImV4cCI6MTY4MzgxNzExNSwiaXNzIjoiaHR0cHM6Ly9hdXRoLWRldi5pbnRvdy50ZWNoIiwiY2xpZW50X2lkIjoiY2xpZW50X2FwcCIsInN1YiI6ImYzZjNiNjlmLWI2NzUtNDJhMC05MzliLTdhMWY0YTM2NWJhOSIsImF1dGhfdGltZSI6MTY4Mzc5NjU1MCwiaWRwIjoibG9jYWwiLCJyb2xlIjpbIk9yZ0FkbWluIiwiT1dVc2VyIiwiU3lzdGVtQWRtaW4iXSwib3JnX2lkIjoiMSIsIm5hbWUiOiJzdGV2ZUBvcmJpdGFsd2l0bmVzcy5jb20iLCJqdGkiOiI4QkY1MDE1OTJFREJCMUFDQzMyODlBQzJCNkYwM0YyNCIsInNpZCI6IkU3M0UzNzcxOEZBN0FFNTc0MEE0NkU3REJCNjExNDZEIiwiaWF0IjoxNjgzODE2NTE1LCJzY29wZSI6WyJvcGVuaWQiLCJjb25zdW1lcl9hcGkiXSwiYW1yIjpbInB3ZCJdfQ.I-DAfohjtMjgdaxrWoHdlHPwucGcxzU7TZ24IxvlTx2eGGsZMl-vg6Xq58MhrqQRFtCJ8AhhkwWdz9IcW63i_CsYGt7bAtJoLpF7nbPm-8gr7hYPcJ9vnaQFsT-1HNwFf2o2jcjnPGqQzelB2gImSad5eMZMi9XDCupYSBkR0TSxicN4__6tvm3bElgTYBV-Pk_NBXrwGcemw_M6ZVY_IKAYg6axecSfOa8vhh9-67vADpQRlROP92OpKEEi7MXJsNMTmKDU9A98VqbeVSaqEFpQKpEqBGjDXZiiy1dDYj_aU_5jug1xDJqXhV2-mEp86KwaukcXFllur4WKp0dY8A',
        documentUrl: '/public/data/test.pdf',
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
            if (this.pdfLoadingTask) {
                // We need to destroy already opened document
                await this.close()
            }

            this.pdfLoadingTask = pdfjsLib.getDocument({
                url: this.documentUrl,
                cMapUrl: CMAP_URL,
                cMapPacked: true,
                maxImageSize: Number.MAX_VALUE,
                disableAutoFetch: true,
                disableStream: true,
                httpHeaders: {
                    Authorization: this.token,
                },
                withCredentials: true,
            })

            this.pdfDocument = await this.pdfLoadingTask.promise

            this.eventBus = new pdfjsViewer.EventBus()
            this.pdfLinkService = new pdfjsViewer.PDFLinkService({
                eventBus: this.eventBus,
            })

            this.l10n = pdfjsViewer.NullL10n

            const container = this.$refs.container
            this.pdfViewer = new pdfjsViewer.PDFViewer({
                container,
                eventBus: this.eventBus,
                linkService: this.pdfLinkService,
                l10n: this.l10n,
                useOnlyCssZoom: true,
                textLayerMode: 2,
            })

            this.pdfLinkService.setViewer(this.pdfViewer)

            this.pdfViewer.setDocument(this.pdfDocument)
        },
    },
})
</script>

<template>
    <section>
        <p>{{ pageNumber }} / {{ totalPages }}</p>
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

.pdf-viewer-container {
    overflow: auto;
    position: absolute;
    width: 100%;
    height: 100%;
}
</style>
