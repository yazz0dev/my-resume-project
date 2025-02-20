// src/components/ContentPreview.vue (UPDATED)
<template>
  <div class="content-preview">
    <div class="preview-actions mb-3">
      <button @click="downloadPDF" class="btn btn-primary me-2">
        <i class="bi bi-download me-2"></i>Download PDF
      </button>
      <button @click="copyToClipboard" class="btn btn-secondary">
        <i class="bi bi-clipboard me-2"></i>Copy HTML
      </button>
    </div>

    <div class="preview-container p-4 bg-white" ref="previewContainer">
      <div :class="templateClass" v-html="finalContent"></div>
    </div>
  </div>
</template>

<script>
import { ref, computed, watch } from 'vue';
import html2pdf from 'html2pdf.js';
import DOMPurify from 'dompurify';

export default {
  name: 'ContentPreview',
  props: {
    contentHtml: {
      type: String,
      required: true
    },
    contentType: {
      type: String,
      required: true
    },
    template: {
      type: String,
      required: true
    },
    contentData: { // Receive form data
      type: Object,
      default: () => ({})
    }
  },
  setup(props) {
    const previewContainer = ref(null);

    const templateClass = computed(() => {
      return `template-${props.template.toLowerCase()}`;
    });

    // Replace placeholders *before* sanitizing
    const processedHtml = computed(() => {
      let html = props.contentHtml;

      if (props.contentType === 'resume') {
        html = html.replace(/\[NAME_PLACEHOLDER\]/g, props.contentData.fullName || '');
        html = html.replace(/\[EMAIL_PLACEHOLDER\]/g, props.contentData.email || '');
        html = html.replace(/\[PHONE_PLACEHOLDER\]/g, props.contentData.phone || '');
        html = html.replace(/\[LINKEDIN_PLACEHOLDER\]/g, props.contentData.linkedin || '');
        html = html.replace(/\[GITHUB_PLACEHOLDER\]/g, props.contentData.github || '');

      }
      // Add placeholder replacement for other content types as needed

      return html;
    });

    // Sanitize *after* placeholder replacement
    const finalContent = computed(() => {
        return DOMPurify.sanitize(processedHtml.value);
    });


    const downloadPDF = async () => {
      const element = previewContainer.value;
      const opt = {
        margin: 1,
        filename: `${props.contentType}-${Date.now()}.pdf`,
        image: { type: 'jpeg', quality: 0.98 },
        html2canvas: { scale: 2 },
        jsPDF: { unit: 'in', format: 'letter', orientation: 'portrait' }
      };

      try {
        await html2pdf().set(opt).from(element).save();
      } catch (error) {
        console.error('PDF generation failed:', error);
      }
    };

    const copyToClipboard = () => {
      const content = previewContainer.value.innerHTML;
      navigator.clipboard.writeText(content)
        .then(() => alert('Content copied to clipboard!'))
        .catch(err => console.error('Failed to copy:', err));
    };

    return {
      previewContainer,
      templateClass,
      finalContent, // Use the sanitized and placeholder-replaced content
      downloadPDF,
      copyToClipboard
    };
  }
};
</script>

<style scoped>
.content-preview {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.preview-container {
  border: 1px solid #ddd;
  border-radius: 4px;
  min-height: 500px;
}
</style>