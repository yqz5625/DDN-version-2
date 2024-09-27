<script setup lang="ts">
import { onMounted, onUnmounted, ref, type Ref } from "vue";
import { EnumCapturedResultItemType, type DSImageData, type OriginalImageResultItem, type Point } from "dynamsoft-core";
import { type NormalizedImageResultItem } from "dynamsoft-document-normalizer";
import { CameraEnhancer, CameraView, QuadDrawingItem, ImageEditorView } from "dynamsoft-camera-enhancer";
import { CapturedResultReceiver, CaptureVisionRouter, type SimplifiedCaptureVisionSettings } from "dynamsoft-capture-vision-router";

let imageEditorViewContainerRef: Ref<HTMLDivElement | null> = ref(null);
let cameraViewContainerRef: Ref<HTMLDivElement | null> = ref(null);
let normalizedImageContainer: Ref<HTMLDivElement | null> = ref(null);
let cameraEnhancer: Ref<Promise<CameraEnhancer> | null> = ref(null);
let router: Ref<Promise<CaptureVisionRouter> | null> = ref(null);
let bShowUiContainer = ref(true);
let bShowImageContainer = ref(false);
let bDisabledBtnEdit = ref(false);
let bDisabledBtnNor = ref(true);
let bShowLoading = ref(true);

let items: Array<any> = [];
let quads: Array<any> = [];
let image: DSImageData;
let confirmTheBoundary: () => void;
let normalize: () => void;

onMounted(async () => {
    try {
        const view = await CameraView.createInstance();
        const dce = await (cameraEnhancer.value = CameraEnhancer.createInstance(view));
        const imageEditorView = await ImageEditorView.createInstance(imageEditorViewContainerRef.value as HTMLDivElement);
        /* Creates an image editing layer for drawing found document boundaries. */
        const layer = imageEditorView.createDrawingLayer();

        /**
         * Creates a CaptureVisionRouter instance and configure the task to detect document boundaries.
         * Also, make sure the original image is returned after it has been processed.
         */
        const normalizer = await (router.value = CaptureVisionRouter.createInstance());

        normalizer.setInput(dce);
        /**
         * Sets the result types to be returned.
         * Because we need to normalize the original image later, here we set the return result type to
         * include both the quadrilateral and original image data.
         */
        const settings =
      '{"CaptureVisionTemplates":[{"ImageROIProcessingNameArray":["roi_default"],"ImageSource":"","MaxParallelTasks":4,"MinImageCaptureInterval":0,"Name":"Default","OutputOriginalImage":0,"SemanticProcessingNameArray":null,"Timeout":10000}],"CharacterModelOptions":[{"DirectoryPath":"","FilterFilePath":"","Name":"NumberLetter"}],"GlobalParameter":{"MaxTotalImageDimension":0},"ImageParameterOptions":[{"BaseImageParameterName":"","BinarizationModes":[{"BinarizationThreshold":-1,"BlockSizeX":5,"BlockSizeY":5,"EnableFillBinaryVacancy":1,"GrayscaleEnhancementModesIndex":-1,"Mode":"BM_LOCAL_BLOCK","MorphOperation":"Close","MorphOperationKernelSizeX":-1,"MorphOperationKernelSizeY":-1,"MorphShape":"Rectangle","ThresholdCompensation":8}],"ColourConversionModes":[{"BlueChannelWeight":-1,"GreenChannelWeight":-1,"Mode":"CICM_GENERAL","RedChannelWeight":-1,"ReferChannel":"H_CHANNEL"}],"GrayscaleEnhancementModes":[{"Mode":"GEM_GENERAL","Sensitivity":-1,"SharpenBlockSizeX":-1,"SharpenBlockSizeY":-1,"SmoothBlockSizeX":-1,"SmoothBlockSizeY":-1}],"GrayscaleTransformationModes":[{"Mode":"GTM_ORIGINAL"},{"Mode":"GTM_INVERTED"}],"IfEraseTextZone":0,"Name":"ip_dlrDefault","RegionPredetectionModes":[{"AspectRatioRange":"[]","FindAccurateBoundary":0,"ForeAndBackgroundColours":"[]","HeightRange":"[]","ImageParameterName":"","MeasuredByPercentage":1,"MinImageDimension":262144,"Mode":"RPM_GENERAL","RelativeRegions":"[]","Sensitivity":1,"SpatialIndexBlockSize":5,"WidthRange":"[]"}],"ScaleDownThreshold":2300,"ScaleUpModes":[{"AcuteAngleWithXThreshold":-1,"LetterHeightThreshold":0,"Mode":"SUM_AUTO","ModuleSizeThreshold":0,"TargetLetterHeight":0,"TargetModuleSize":0}],"TextDetectionMode":{"CharHeightRange":[20,1000,1],"Direction":"HORIZONTAL","MaxSpacingInALine":-1,"Mode":"TTDM_LINE","Sensitivity":7},"TextureDetectionModes":[{"Mode":"TDM_GENERAL_WIDTH_CONCENTRATION","Sensitivity":5}]}],"LabelRecognizerTaskSettingOptions":[{"BaseLabelRecognizerTaskSettingName":"","DictionaryCorrectionThresholds":[{"MaxWordLength":256,"MinWordLength":3,"Threshold":1}],"DictionaryPath":"","MaxThreadsInOneTask":4,"Name":"dlr_task_default","SectionImageParameterArray":[{"ContinueWhenPartialResultsGenerated":1,"ImageParameterName":"ip_dlrDefault","Section":"ST_REGION_PREDETECTION"},{"ContinueWhenPartialResultsGenerated":1,"ImageParameterName":"ip_dlrDefault","Section":"ST_TEXT_LINE_LOCALIZATION"},{"ContinueWhenPartialResultsGenerated":1,"ImageParameterName":"ip_dlrDefault","Section":"ST_TEXT_LINE_RECOGNITION"}],"StartSection":"ST_REGION_PREDETECTION","StringLengthRange":[3,10],"StringRegExPattern":"^[0-9]+$","TerminateSetting":{"Section":"ST_NULL","Stage":"IRUT_NULL"},"TextLineSpecificationNameArray":["tls_default"]}],"TargetROIDefOptions":[{"BaseTargetROIDefName":"","Location":{"Offset":{"FirstPoint":[0,0],"FourthPoint":[0,100],"MeasuredByPercentage":1,"ReferenceObjectOriginIndex":0,"ReferenceObjectSizeType":"default","SecondPoint":[100,0],"ThirdPoint":[100,100]}},"Name":"roi_default","PauseFlag":0,"TaskSettingNameArray":["dlr_task_default"]}],"TextLineSpecificationOptions":[{"ApplicableTextLineNumbers":"","BaseTextLineSpecificationName":"","BinarizationModes":[{"BinarizationThreshold":-1,"BlockSizeX":5,"BlockSizeY":5,"EnableFillBinaryVacancy":1,"GrayscaleEnhancementModesIndex":-1,"Mode":"BM_LOCAL_BLOCK","MorphOperation":"Erode","MorphOperationKernelSizeX":-1,"MorphOperationKernelSizeY":-1,"MorphShape":"Rectangle","ThresholdCompensation":10}],"CharHeightRange":[20,1000,1],"CharacterModelName":"NumberLetter","CharacterNormalizationModes":[{"Mode":"CNM_AUTO","MorphArgument":"","MorphOperation":"Erode"}],"GrayscaleEnhancementModes":[{"Mode":"GEM_GENERAL","Sensitivity":-1,"SharpenBlockSizeX":-1,"SharpenBlockSizeY":-1,"SmoothBlockSizeX":-1,"SmoothBlockSizeY":-1}],"Name":"tls_default","StringLengthRange":[0,11],"StringRegExPattern":""}]}';
        await router.initSettings(settings);
        // let newSettings = await normalizer.getSimplifiedSettings("DetectDocumentBoundaries_Default");
        // newSettings.capturedResultItemTypes |= EnumCapturedResultItemType.CRIT_ORIGINAL_IMAGE;
        // await normalizer.updateSettings("DetectDocumentBoundaries_Default", newSettings);
        cameraViewContainerRef.value!.append(view.getUIElement());

        /* Defines the result receiver for the task.*/
        const resultReceiver = new CapturedResultReceiver();
        resultReceiver.onCapturedResultReceived = (result) => {
            const originalImage = result.items.filter(item => item.type === EnumCapturedResultItemType.CRIT_ORIGINAL_IMAGE);
            if (originalImage.length) {
                image = (originalImage[0] as OriginalImageResultItem).imageData;
            }
            items = result.items.filter(item => item.type === EnumCapturedResultItemType.CRIT_DETECTED_QUAD);
        }
        normalizer.addResultReceiver(resultReceiver);

        confirmTheBoundary = () => {
            if (!dce.isOpen() || !items.length) return;
            /* Hides the cameraView and shows the imageEditorView. */
            bShowUiContainer.value = false
            bShowImageContainer.value = true;
            /* Draws the image on the imageEditorView first. */
            imageEditorView.setOriginalImage(image);
            quads = [];
            /* Draws the document boundary (quad) over the image. */
            for (let i = 0; i < items.length; i++) {
                if (items[i].type === EnumCapturedResultItemType.CRIT_ORIGINAL_IMAGE) continue;
                const points = items[i].location.points;
                const quad = new QuadDrawingItem({ points });
                quads.push(quad);
                layer.addDrawingItems(quads);
            }
            bDisabledBtnNor.value = false;
            bDisabledBtnEdit.value = true;
            normalizer.stopCapturing();
        }

        normalize = async () => {
            /* Get the selected quadrilateral */
            let seletedItems = imageEditorView.getSelectedDrawingItems();
            let quad;
            if (seletedItems.length) {
                quad = (seletedItems[0] as QuadDrawingItem).getQuad();
            } else {
                quad = items[0].location;
            }
            const isPointOverBoundary = (point: Point) => {
                if(point.x < 0 || 
                point.x > image.width || 
                point.y < 0 ||
                point.y > image.height) {
                    return true;
                } else {
                    return false;
                }
            };
            /* Check if the points beyond the boundaries of the image. */
            if (quad.points.some((point: Point) => isPointOverBoundary(point))) {
                alert("The document boundaries extend beyond the boundaries of the image and cannot be used to normalize the document.");
                return;
            }
            
            /* Hides the imageEditorView. */
            bShowImageContainer.value = false;
            /* Removes the old normalized image if any. */
            normalizedImageContainer.value!.innerHTML = "";
            /**
             * Sets the coordinates of the ROI (region of interest)
             * in the built-in template "normalize-document".
             */
            let newSettings = await normalizer.getSimplifiedSettings("normalize-document") as SimplifiedCaptureVisionSettings;
            newSettings.roiMeasuredInPercentage = false;
            newSettings.roi.points = quad.points;
            await normalizer.updateSettings("normalize-document", newSettings);
            /* Executes the normalization and shows the result on the page */
            let normalizeResult = await normalizer.capture(image, "normalize-document");
            if (normalizeResult.items[0]) {
                normalizedImageContainer.value!.append((normalizeResult.items[0] as NormalizedImageResultItem).toCanvas());
            }
            layer.clearDrawingItems();
            bDisabledBtnNor.value = true;
            bDisabledBtnEdit.value = false;
            /* show video view */
            bShowUiContainer.value = true
            view.getUIElement().style.display = "";
            await normalizer.startCapturing("Default");
        }

        await dce.open();
        /* Uses the built-in template "DetectDocumentBoundaries_Default" to start a continuous boundary detection task. */
        await normalizer.startCapturing("Default");
        bShowLoading.value = false;
    } catch (ex: any) {
        let errMsg = ex.message || ex;
        console.error(errMsg);
        alert(errMsg);
    }
})

onUnmounted(async () => {
    (await router.value)?.dispose();
    (await cameraEnhancer.value)?.dispose();
    console.log('VideoNormalizer Component Unmount');
})
</script>

<template>
    <div id="div-loading" v-show="bShowLoading">Loading...</div>
    <div id="div-video-btns">
        <button id="confirm-quad-for-normalization" @click="confirmTheBoundary" :disabled="bDisabledBtnEdit">Confirm the
            Boundary</button>
        <button id="normalize-with-confirmed-quad" @click="normalize" :disabled="bDisabledBtnNor">Normalize</button>
    </div>
    <div id="div-ui-container" style="margin-top: 10px;height: 500px;" ref="cameraViewContainerRef"
        v-show="bShowUiContainer"></div>
    <div id="div-image-container" style="display:none; width: 100vw; height: 70vh" ref="imageEditorViewContainerRef"
        v-show="bShowImageContainer">
        <div class="dce-image-container" style="width: 100%; height: 100%"></div>
    </div>
    <div id="normalized-result" ref="normalizedImageContainer"></div>
</template>
    
<style scoped>
#div-video-btns {
    width: 75%;
    margin: 0 auto;
    margin-bottom: 10px;
    display: flex;
    justify-content: space-around;
}
</style>
