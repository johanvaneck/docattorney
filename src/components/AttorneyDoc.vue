<template>
    <article class="flex flex-col gap-4">

        <Panel header="Variables" :toggleable="true">
            <section v-for="section in sections" :key="section.label">
                <h1 v-if="section.label" class="text-xl py-8">{{ section.label }}</h1>
                <div class="grid sm:grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
                    <div v-for="item in section.items" :key="item.label" class="flex flex-col gap-2">
                        <label :for="item.label">{{ item.label }}</label>
                        <InputText :id="item.label" v-model="item.text" :aria-describedby="`${item.label}-help`" />
                        <small v-if="item.description" :id="`${item.label}-help`">{{ item.description }}</small>
                    </div>
                </div>
            </section>
        </Panel>

        <Panel header="PDF Preview" :toggleable="true">
            <iframe class="w-full h-screen" :src="currentPdf?.url ?? ''" frameborder="0"></iframe>
        </Panel>

        <Panel header="LOD Template" :toggleable="true">
            <template #icons>
                <Button severity="secondary" rounded label="Copy template" @click="copyLod" />
            </template>
            <pre class="overflow-auto"> {{ lodTemplate }} </pre>
        </Panel>
    </article>
</template>

<script setup lang="ts">
import InputText from 'primevue/inputtext';
import Textarea from 'primevue/textarea';
import Panel from 'primevue/panel';
import Button from 'primevue/button';
import { computed, onMounted, reactive, ref, watch } from 'vue';
import { jsPDF } from 'jspdf';
import logoUrl from '@/assets/image.png';

// FILE REFERENCE:

// CLIENT’S DETAILS:
// NAME & SURNAME/COMPANY NAME:
// ID/REG NO:

// CLIENT’S ADDRESS LINE 1
// CLIENT’S ADDRESS LINE 2
// CLIENT’S ADDRESS LINE 3
// SPECIFY ADDRESS
// CLIENT’S EMAIL ADDRESS
// DEBTOR'S DETAILS:
// NAME & SURNAME/COMPANY NAME:
// ID/REG NO:

// ADDRESS LINE 1
// ADDRESS LINE 2
// ADDRESS LINE 3
// SPECIFY ADDRESS
// DEBTOR’S EMAIL ADDRESS:

// CORRESPONDENTS DETAILS (ALWAYS CAPS):
// NAME:
// ADDRESS: LINE 1
// LINE 2
// LINE 3
// Reference:
// CORRESPONDENT’S EMAIL
// LOD DETAILS:
// CAUSE OF ACTION (I.e goods sold and delivered…)
// CLAIM AMOUNT (*numerical digits only with decimals)
// INTEREST RATE

// COURT DETAILS:
// COURT DISTRICT
// COURT HELD AT
// COURT ADDRESS


// SHERIFF'S DETAILS
// SHERIFF'S NAME

// SHERIFF ADDRESS LINE 2
// SHERIFF ADDRESS LINE 2
// SHERIFF ADDRESS LINE 3


// REQUEST FOR DEFAULT JUDGMENT DETAILS:
// CASE NUMBER:
// DATE OF SERVICE (DD/MM/YYYY):
// RETURN OF SERVICE CHARGE (*numerical digits only with decimals):

// SEC 65 DETAILS
// DATE JUDGMENT GRANTED (DD/MM/YYYY)

enum LabelText {
    FileReference = 'File Reference',
    ClientName = 'Name & Surname/Company Name',
    ClientId = 'ID/Reg No',
    ClientAddress1 = 'Client’s Address Line 1',
    ClientAddress2 = 'Client’s Address Line 2',
    ClientAddress3 = 'Client’s Address Line 3',
    ClientSpecifyAddress = 'Specify Address',
    ClientEmail = 'Client’s Email Address',
    DebtorName = 'Name & Surname/Company Name',
    DebtorId = 'ID/Reg No',
    DebtorAddress1 = 'Address Line 1',
    DebtorAddress2 = 'Address Line 2',
    DebtorAddress3 = 'Address Line 3',
    DebtorSpecifyAddress = 'Specify Address',
    DebtorEmail = 'Debtor’s Email Address',
    CorrespondentName = 'Name',
    CorrespondentAddress1 = 'Address Line 1',
    CorrespondentAddress2 = 'Address Line 2',
    CorrespondentAddress3 = 'Address Line 3',
    CorrespondentReference = 'Reference',
    CorrespondentEmail = 'Correspondent’s Email',
    LodCauseOfAction = 'Cause of Action',
    LodClaimAmount = 'Claim Amount',
    LodInterestRate = 'Interest Rate',
    CourtDistrict = 'Court District',
    CourtHeldAt = 'Court Held At',
    CourtAddress = 'Court Address',
    SheriffName = 'Sheriff’s Name',
    SheriffAddress1 = 'Sheriff Address Line 1',
    SheriffAddress2 = 'Sheriff Address Line 2',
    SheriffAddress3 = 'Sheriff Address Line 3',
    RequestCaseNumber = 'Case Number',
    RequestDateOfService = 'Date of Service (DD/MM/YYYY)',
    RequestReturnOfServiceCharge = 'Return of Service Charge',
    Sec65DateJudgmentGranted = 'Date Judgment Granted (DD/MM/YYYY)',
}

const currentPdf = ref<{ pdf: jsPDF, url: string }>();
const sections = reactive<Array<{
    label?: string;
    items: Array<{
        label: LabelText;
        text: string;
        description?: string;
    }>;
}>>([
    {
        label: 'File Reference',
        items: [
            { label: LabelText.FileReference, text: '[MY_UNIQUE_FILE_REFERENCE]' },
        ]
    },
    // {
    //     label: 'Client’s Details',
    //     items: [
    //         { label: LabelText.ClientName, text: '' },
    //         { label: LabelText.ClientId, text: '' },
    //         { label: LabelText.ClientAddress1, text: '' },
    //         { label: LabelText.ClientAddress2, text: '' },
    //         { label: LabelText.ClientAddress3, text: '' },
    //         { label: LabelText.ClientSpecifyAddress, text: '' },
    //         { label: LabelText.ClientEmail, text: '' },
    //     ],
    // },
    {
        label: 'Debtor’s Details',
        items: [
            { label: LabelText.DebtorName, text: '[MY_DEBTOR_NAME]' },
            // { label: LabelText.DebtorId, text: '' },
            { label: LabelText.DebtorAddress1, text: '[MY_DEBTOR_ADDRESS_LINE_1]' },
            { label: LabelText.DebtorAddress2, text: '[MY_DEBTOR_ADDRESS_LINE_2]' },
            { label: LabelText.DebtorAddress3, text: '[MY_DEBTOR_ADDRESS_LINE_3]' },
            // { label: LabelText.DebtorSpecifyAddress, text: '' },
            // { label: LabelText.DebtorEmail, text: '' },
        ],
    },
    // {
    //     label: 'Correspondents Details',
    //     items: [
    //         { label: LabelText.CorrespondentName, text: '' },
    //         { label: LabelText.CorrespondentAddress1, text: '' },
    //         { label: LabelText.CorrespondentAddress2, text: '' },
    //         { label: LabelText.CorrespondentAddress3, text: '' },
    //         { label: LabelText.CorrespondentReference, text: '' },
    //         { label: LabelText.CorrespondentEmail, text: '' },
    //     ],
    // },
    {
        label: 'Lod Details',
        items: [
            // { label: LabelText.LodCauseOfAction, text: '' },
            { label: LabelText.LodClaimAmount, text: '[MY_LOD_CLAIM_AMOUNT]' },
            // { label: LabelText.LodInterestRate, text: '' },
        ],
    },
    // {
    //     label: 'Court Details',
    //     items: [
    //         { label: LabelText.CourtDistrict, text: '' },
    //         { label: LabelText.CourtHeldAt, text: '' },
    //         { label: LabelText.CourtAddress, text: '' },
    //     ],
    // },
    // {
    //     label: 'Sheriff’s Details',
    //     items: [
    //         { label: LabelText.SheriffName, text: '' },
    //         { label: LabelText.SheriffAddress1, text: '' },
    //         { label: LabelText.SheriffAddress2, text: '' },
    //         { label: LabelText.SheriffAddress3, text: '' },
    //     ],
    // }
]);


function getTextFromLabel(label: LabelText) {
    const text = sections.flatMap(section => section.items).find(item => item.label === label)?.text;
    if (text === '' || text === undefined) {
        return '#########'
    }
    return text;
}

const lodTemplate = computed(() => `
Aan / to: ${getTextFromLabel(LabelText.DebtorName)}
Datum / Date: ${new Date().toLocaleDateString()}

Via:   
${getTextFromLabel(LabelText.DebtorAddress1)}
${getTextFromLabel(LabelText.DebtorAddress2)}
${getTextFromLabel(LabelText.DebtorAddress3)}

Per Geregistreerde Pos / Per Registered Post

Ons verw / our ref: P VENTER/${getTextFromLabel(LabelText.FileReference)}

U verw / your ref: ${getTextFromLabel(LabelText.FileReference)}

Dear Sir / Madam,

JACOBS FOURIE INC // ${getTextFromLabel(LabelText.DebtorName)}

1. We refer to the above mentioned matter and confirm we act herein on behalf of our client, JACOBS FOURIE INC.

2. We have been instructed by our client to demand from you, as we hereby do, immediate payment of the sum of R${getTextFromLabel(LabelText.LodClaimAmount)} due to our client in respect of goods sold and delivered and/or services rendered at your special request and instance.

3. Payment must be deposit into our bank account, namely:

        JACOBS FOURIE ATTORNEYS TRUST ACCOUNT
        ABSA BANK
        ACCOUNT NUMBER:  40-8047-4732
        BRANCH CODE:  632005
        REF: ${getTextFromLabel(LabelText.FileReference)}

4. Kindly take note that payment must reach our offices within 7 (SEVEN) days from the date hereof failing which we shall proceed with Legal steps against you without further notice.

5. We trust that the above mentioned actions may not be necessary and look forward to proof of payment of the account or reasonable settlement arrangements of the amount outstanding.

Yours faithfully,

PIERRE POKUREUR
JACOBS FOURIE INC.
info@jacobsfourie.co.za
`)

function copyLod() {
    navigator.clipboard.writeText(lodTemplate.value);
}

function generatePdf() {
    // Default export is a4 paper, portrait, using millimeters for units
    const doc = new jsPDF();

    const maxWidth = 180; // Maximum width of the text in mm
    const fontSize = 12; // Desired font size

    // Set the font size
    doc.setFontSize(fontSize);

    const padding = 10;
    const s = 40;
    const imgWidth = s;
    const imgHeight = s;
    // Add company logo
    doc.addImage(logoUrl, 'PNG', (maxWidth - imgWidth + padding), padding, imgWidth, imgHeight); // Adjust the position and size as needed

    // Split text into lines that fit within the specified width
    const lines = doc.splitTextToSize(lodTemplate.value, maxWidth);

    // Add the lines to the PDF document
    doc.text(lines, 10, 10);

    // Revoke the object URL to free up memory
    if (currentPdf.value?.url) {
        URL.revokeObjectURL(currentPdf.value.url);
    }
    currentPdf.value = {
        pdf: doc,
        url: URL.createObjectURL(doc.output('blob')),
    }
    // doc.save("docattorney.pdf");
}

let timeout: number;
function debounce(func: Function, wait: number = 1000) {
    const later = () => {
        clearTimeout(timeout);
        func();
    };
    clearTimeout(timeout);
    timeout = setTimeout(later, wait);
}

watch(lodTemplate, () => {
    debounce(generatePdf);
});

onMounted(() => {
    debounce(generatePdf);
});
</script>
