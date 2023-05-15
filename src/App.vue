<script setup>
import { ref, computed } from "vue";
import { Buffer } from "buffer";
import * as fontkit from "fontkit";
import googleFonts from "./google-fonts.json";
import BsNavbar from "./components/BsNavbar.vue";
import BsSelect from "./components/BsSelect.vue";
import BsInputRange from "./components/BsInputRange.vue";
import BsInputText from "./components/BsInputText.vue";
import BsOffcanvas from "./components/BsOffcanvas.vue";

const DEFAULT_FONT_FAMILY = "Roboto";
const DEFAULT_FALLBACK_FONT_FAMILY = "Arial";
const DEFAULT_FONT_STYLE = "regular";
const DEFAULT_FONT_SIZE = 30;
const DEFAULT_SIZE_ADJUST = 100;
const WEB_SAFE_FONTS = [
  "Arial",
  "Verdana",
  "Tahoma",
  "Trebuchet MS",
  "Times New Roman",
  "Georgia",
  "Garamond",
  "Courier New",
  "Brush Script MT",
];
const GOOGLE_FONTS = googleFonts.items.reduce((acc, font) => {
  const { family, files } = font;
  acc[family] = files;
  return acc;
}, {});
const DEFAULT_SENTENCE = `Lorem ipsum dolor sit amet, consectetur adipiscing elit, sed do eiusmod
tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam,
quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo
consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse
cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat
non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.`;

function generateFontOverrides(fontPath) {
  var xhr = new XMLHttpRequest();
  xhr.open("GET", fontPath, true);
  xhr.responseType = "arraybuffer";
  xhr.onload = function (e) {
    if (this.status == 200) {
      const buffer = new Buffer(this.response);
      const font = fontkit.create(buffer);
      const formatForCSS = (x) => `${Math.abs(x * 100)}`;
      ascentOverride.value = formatForCSS(font.ascent / font.unitsPerEm);
      descentOverride.value = formatForCSS(font.descent / font.unitsPerEm);
      lineGapOverride.value = formatForCSS(font.lineGap / font.unitsPerEm);
    }
  };
  xhr.send();
}

function ucfirst(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

function matchFontWeight(weight) {
  switch (parseInt(weight, 10)) {
    case 100:
      return "Thin";
    case 200:
      return "ExtraLight";
    case 300:
      return "Light";
    case 400:
      return "Regular";
    case 500:
      return "Medium";
    case 600:
      return "SemiBold";
    case 700:
      return "Bold";
    case 800:
      return "ExtraBold";
    case 900:
      return "Black";
  }
}

const sentence = ref("");

const fontFamily = ref(DEFAULT_FONT_FAMILY);
const fallbackFontFamily = ref(DEFAULT_FALLBACK_FONT_FAMILY);
const fontVariant = ref(DEFAULT_FONT_STYLE);
const fontSize = ref(DEFAULT_FONT_SIZE);

const ascentOverride = ref();
const descentOverride = ref();
const lineGapOverride = ref();
const sizeAdjust = ref(DEFAULT_SIZE_ADJUST);

const isAndroid = /Android/i.test(navigator.userAgent);

const computedSentece = computed(() => {
  return sentence.value !== "" ? sentence.value : DEFAULT_SENTENCE;
});

const fontVariants = computed(() => {
  return Object.keys(GOOGLE_FONTS[fontFamily.value]).reduce((acc, variant) => {
    const { groups } = variant.match(
      /^(?<weight>[0-9]+|regular)?(?<style>[a-z]+)?$/i
    );
    const weight =
      groups.weight && !isNaN(groups.weight)
        ? `${matchFontWeight(groups.weight)} ${groups.weight}`
        : "Regular 400";
    const style = groups.style ? ucfirst(groups.style) : "";

    acc[variant] = {
      option: `${weight} ${style}`.trim(),
      weight: isNaN(groups.weight) ? 400 : groups.weight,
      style: groups.style ?? "normal",
    };
    return acc;
  }, {});
});

const fontWeight = computed(() => {
  return fontVariants.value[fontVariant.value]?.weight;
});

const fontStyle = computed(() => {
  return fontVariants.value[fontVariant.value]?.style;
});

const fontFamilyFile = computed(() => {
  if (Object.keys(GOOGLE_FONTS[fontFamily.value]).includes(fontVariant.value)) {
    const file = GOOGLE_FONTS[fontFamily.value][fontVariant.value];
    generateFontOverrides(file);
    return file;
  } else {
    const [file] = Object.values(GOOGLE_FONTS[fontFamily.value]);
    generateFontOverrides(file);
    return file;
  }
});

const googleFontStyleCSS = computed(() => {
  return `@font-face {
  font-family: "${fontFamily.value}";
  font-weight: ${fontWeight.value};
  font-style: ${fontStyle.value};
  font-display: swap;
  src: url(${fontFamilyFile.value})
    format("woff2");
}`;
});

const fallbackFontStyleCSS = computed(() => {
  return `@font-face {
  font-family: "FallbackFont";
  src: local("${fallbackFontFamily.value}"), local(system-ui);
  ascent-override: ${ascentOverride.value}%;
  descent-override: ${descentOverride.value}%;
  line-gap-override: ${lineGapOverride.value}%;
  size-adjust: ${sizeAdjust.value}%;
}`;
});
</script>

<template>
  <component is="style" v-text="googleFontStyleCSS" />
  <component is="style" v-text="fallbackFontStyleCSS" />
  <bs-navbar />
  <main>
    <div class="border-bottom">
      <div class="container">
        <div
          class="row flex-column flex-md-row gap-4 gap-lg-0 py-5 position-relative"
        >
          <div
            class="col col-12 col-lg-6 d-flex align-items-center gap-2 col-auto"
          >
            <bs-select
              id="google-font-family"
              class="form-select--font"
              label="Google Font"
              placeholder="Choose a font"
              v-model="fontFamily"
            >
              <option
                v-for="(_, font) in GOOGLE_FONTS"
                :value="font"
                :key="font"
                :selected="font === fontFamily"
                v-text="font"
              />
            </bs-select>
            <img src="./assets/arrow-left-right-line.svg" width="20" />
            <bs-select
              id="fallback-font-family"
              class="form-select--font"
              label="Fallback Font"
              placeholder="Choose a font"
              v-model="fallbackFontFamily"
              :disabled="isAndroid"
            >
              <option
                v-for="font in WEB_SAFE_FONTS"
                :value="font"
                :key="font"
                :selected="font === fallbackFontFamily"
                v-text="font"
              />
            </bs-select>
          </div>
          <div class="col col-lg-3">
            <bs-select
              id="font-variant"
              label="Variants"
              placeholder="Choose a variant"
              v-model="fontVariant"
            >
              <option
                v-for="(variant, key) in fontVariants"
                :value="key"
                :key="key"
                :selected="key === fontVariant"
                v-text="variant.option"
              />
            </bs-select>
          </div>
          <div class="col col-lg-3 d-flex align-items-center">
            <bs-input-range
              v-model="sizeAdjust"
              :min="0"
              :max="200"
              :step="0.1"
              label="Size Adjust"
            >
              <template #rangeHint> {{ sizeAdjust }}% </template>
            </bs-input-range>
          </div>
          <button
            class="btn btn-light position-absolute top-100 end-0 translate-middle w-auto rounded-circle collapsed"
            type="button"
            data-bs-toggle="collapse"
            data-bs-target="#preview-settings"
            aria-expanded="false"
            aria-controls="preview-settings"
          >
            <img src="./assets/arrow-down-line.svg" width="24" />
          </button>
        </div>
      </div>
    </div>

    <div class="collapse border-bottom" id="preview-settings">
      <div class="container">
        <div class="row flex-column flex-md-row gap-4 gap-md-0 py-5">
          <div class="col col-md-8">
            <bs-input-text
              v-model="sentence"
              id="sentence"
              label="Type here to preview text"
              placeholder="Type here to preview text"
            />
          </div>
          <div class="col col-md-4 d-flex align-items-center">
            <bs-input-range
              v-model="fontSize"
              :min="8"
              :max="300"
              :step="1"
              label="Font Size"
            >
              <template #rangeHint> {{ fontSize }}px </template>
            </bs-input-range>
          </div>
        </div>
      </div>
    </div>
    <div class="container py-5">
      <div class="preview position-relative">
        <article
          v-text="computedSentece"
          :style="{
            'font-size': `${fontSize}px`,
            'font-family': `'${fontFamily}'`,
            'font-style': `${fontStyle}`,
            'font-weight': `${fontWeight}`,
          }"
        />
        <article
          class="fallback-preview position-absolute top-0 start-0 opacity-25"
          v-text="computedSentece"
          :style="{
            'font-size': `${fontSize}px`,
            'font-family': `'FallbackFont'`,
            'font-style': `${fontStyle}`,
            'font-weight': `${fontWeight}`,
          }"
        />
      </div>
    </div>
  </main>
  <bs-offcanvas id="offcanvas" title="CSS code snippet">
    <p>
      To embed this fallback font, copy the code below into the
      <code>&lt;head&gt;</code> of your html
    </p>
    <pre
      v-text="fallbackFontStyleCSS"
      class="rounded bg-light p-4 small mb-4"
    />
    <p>CSS rules to specify families</p>
    <pre
      class="rounded bg-light p-4 small mb-4"
      v-text="`font-family: '${fontFamily}', 'FallbackFont';`"
    />
  </bs-offcanvas>
</template>

<style scoped lang="scss">
.preview {
  article {
    word-break: break-word;
  }
}
[data-bs-toggle="collapse"] {
  aspect-ratio: 1;
  img {
    transition: all 0.2s;
  }
  &:not(.collapsed) img {
    transform: rotate(180deg);
  }
}
.form-select--font {
  flex: 1 1 0;
}
</style>
