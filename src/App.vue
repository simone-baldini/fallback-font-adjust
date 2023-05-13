<script setup>
import { ref, computed } from "vue";
import { Buffer } from "buffer";
import * as fontkit from "fontkit";
import googleFonts from "./google-fonts.json";

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
      const formatForCSS = (x) => `${Math.abs(x * 100)}%`;
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
  src: local("${fallbackFontFamily.value}");
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
  <nav class="navbar border-bottom py-3 px-md-3">
    <div class="container-fluid">
      <a class="navbar-brand d-flex align-items-center me-auto" href="#">
        <img src="./assets/quill-pen-line.svg" width="24" class="me-2" />
        Fallback Font Adjust
      </a>
      <!-- <a class="ms-2" href="#">
        <img src="./assets/github-fill.svg" width="32" />
      </a> -->
      <a
        class="btn btn-outline-light border-white"
        data-bs-toggle="offcanvas"
        href="#offcanvas"
        role="button"
        aria-controls="offcanvas"
      >
        <img src="./assets/code-line.svg" width="28" />
      </a>
    </div>
  </nav>
  <main>
    <div class="border-bottom">
      <div class="container">
        <div class="row flex-column flex-md-row gap-4 gap-lg-0 py-5">
          <div
            class="col col-12 col-lg-6 d-flex align-items-center gap-2 col-auto"
          >
            <div class="form-floating">
              <select
                class="form-select rounded-pill ps-4 pe-5"
                id="google-font-family"
                aria-label="Choose a font"
                v-model="fontFamily"
              >
                <option selected>Choose a font</option>
                <option
                  v-for="(_, font) in GOOGLE_FONTS"
                  :value="font"
                  :key="font"
                  v-text="font"
                />
              </select>
              <label for="google-font-family" class="opacity-75 ps-4 pe-5">
                Google Font
              </label>
            </div>
            <img src="./assets/arrow-left-right-line.svg" width="20" />
            <div class="form-floating">
              <select
                class="form-select rounded-pill ps-4 pe-5"
                id="fallback-font-family"
                aria-label="Choose a font"
                v-model="fallbackFontFamily"
              >
                <option selected>Choose a font</option>
                <option
                  v-for="font in WEB_SAFE_FONTS"
                  :value="font"
                  :key="font"
                  v-text="font"
                />
              </select>
              <label for="fallback-font-family" class="opacity-75 ps-4 pe-5">
                Fallback Font
              </label>
            </div>
          </div>
          <div class="col col-lg-3">
            <div class="form-floating">
              <select
                class="form-select rounded-pill ps-4 pe-5"
                id="font-variant"
                aria-label=">Choose a variant"
                v-model="fontVariant"
              >
                <option selected>Choose a variant</option>
                <option
                  v-for="(variant, key) in fontVariants"
                  :value="key"
                  :key="key"
                  v-text="variant.option"
                />
              </select>
              <label for="font-variant" class="opacity-75 ps-4 pe-5">
                Variants
              </label>
            </div>
          </div>
          <div class="col col-lg-3 d-flex align-items-center">
            <div
              class="form-range"
              :style="`
          --progress: ${((parseInt(sizeAdjust, 10) + 300) * 100) / 600}%;
          --offset: ${
            10 - (((parseInt(sizeAdjust, 10) + 300) * 100) / 600) * 0.2
          }px;
          `"
            >
              <label class="opacity-75">Size Adjust</label>
              <span class="rangeHint" v-text="`${sizeAdjust}%`" />
              <input
                type="range"
                v-model="sizeAdjust"
                min="-300"
                max="300"
                step="0.1"
              />
            </div>
          </div>
          <div class="col col-auto d-flex align-items-center">
            <button
              class="btn btn-outline-light border-white"
              type="button"
              data-bs-toggle="collapse"
              data-bs-target="#preview-settings"
              aria-expanded="false"
              aria-controls="preview-settings"
            >
              <img src="./assets/equalizer-fill.svg" width="24" />
            </button>
          </div>
        </div>
      </div>
    </div>
    <div class="collapse border-bottom" id="preview-settings">
      <div class="container">
        <div class="row flex-column flex-md-row gap-4 gap-md-0 py-5">
          <div class="col col-md-8">
            <div class="form-floating">
              <input
                type="text"
                class="form-control rounded-pill px-4"
                id="sentence"
                placeholder="Type here to preview text"
                v-model="sentence"
              />
              <label for="sentence" class="opacity-75 px-4">
                Type here to preview text
              </label>
            </div>
          </div>
          <div class="col col-md-4 d-flex align-items-center">
            <div
              class="form-range"
              :style="`
          --progress: ${((parseInt(fontSize, 10) - 8) * 100) / 292}%;
          --offset: ${
            10 - (((parseInt(fontSize, 10) - 8) * 100) / 292) * 0.2
          }px;
          `"
            >
              <label class="opacity-75">Font Size</label>
              <span class="rangeHint" v-text="`${fontSize}px`" />
              <input
                type="range"
                v-model="fontSize"
                min="8"
                max="300"
                step="1"
              />
            </div>
            <!-- <div class="dropdown">
              <button
                class="btn btn-outline-light border-white text-body dropdown-toggle rangeHint me-2"
                type="button"
                data-bs-toggle="dropdown"
                aria-expanded="false"
                v-text="`${fontSize}px`"
              />
              <ul class="dropdown-menu">
                <li
                  v-for="size in [
                    8, 12, 14, 20, 24, 32, 40, 64, 96, 120, 184, 280,
                  ]"
                  :key="size"
                  :value="size"
                >
                  <button
                    class="dropdown-item"
                    v-text="size"
                    @click="fontSize = size"
                  />
                </li>
              </ul>
            </div>
            <input
              type="range"
              class="flex-grow-1"
              v-model="fontSize"
              min="8"
              max="300"
              step="1"
              :style="`--progress: ${
                ((parseInt(fontSize, 10) - 8) * 100) / 292
              }%`"
            /> -->
          </div>
        </div>
      </div>
    </div>
    <div class="container py-5">
      <article
        v-text="computedSentece"
        :data-content="computedSentece"
        :style="{
          'font-size': `${fontSize}px`,
          'font-family': `'${fontFamily}'`,
          'font-style': `${fontStyle}`,
          'font-weight': `${fontWeight}`,
        }"
      />
    </div>
  </main>
  <div
    class="offcanvas offcanvas-end"
    tabindex="-1"
    id="offcanvas"
    data-bs-backdrop="false"
    aria-labelledby="offcanvas"
  >
    <div class="offcanvas-header border-bottom">
      <h5 class="offcanvas-title" id="offcanvas">CSS code snippet</h5>
      <button
        type="button"
        class="btn-close"
        data-bs-dismiss="offcanvas"
        aria-label="Close"
      ></button>
    </div>
    <div class="offcanvas-body py-4">
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
    </div>
  </div>
</template>

<style scoped lang="scss">
article {
  position: relative;
  &::before {
    content: attr(data-content);
    font-family: "FallbackFont";
    position: absolute;
    inset: 0;
    opacity: 0.3;
  }
}

.rangeHint {
  min-width: 5rem;
  text-align: center;
}

.form-range {
  position: relative;
  padding: 1.625rem 0 0.625rem;
  height: calc(3.5rem + 2px);
  line-height: 1.25;
  label {
    transform: scale(0.85) translateY(-0.5rem) translateX(0.15rem);
    transform-origin: 0 0;
    position: absolute;
    top: 0;
    left: 0;
    padding: 1rem 0;
  }
  input[type="range"] {
    width: 100%;
  }
  span {
    content: attr(data-value);
    position: absolute;
    top: -0.75rem;
    padding: 0.5rem 0.75rem;
    font-size: 80%;
    background-color: #1a73e8;
    color: #fff;
    border-radius: 50rem;
    left: 148px;
    transform: scale(0) translateX(-50%);
    transform-origin: left;
    transition: transform 0.2s ease-in-out, opacity 0.2s ease-in-out;
    opacity: 0;
    left: calc(var(--progress) + var(--offset));
    &::before {
      content: " ";
      width: 0;
      height: 0;
      border-left: 4px solid transparent;
      border-right: 4px solid transparent;
      border-top: 4px solid #1a73e8;
      position: absolute;
      bottom: 0;
      left: 50%;
      transform: translate(-50%, 100%);
    }
  }
  &:focus-within {
    span {
      opacity: 1;
      transform: scale(1) translateX(-50%);
    }
  }
}

input[type="range"] {
  -webkit-appearance: none;
  appearance: none;
  background-color: #d2e3fc;
  height: 0.25rem;
  border-radius: 50rem;
  cursor: pointer;
}

/***** Chrome, Safari, Opera, and Edge Chromium *****/
input[type="range"]::-webkit-slider-runnable-track {
  border-radius: 50rem;
  background-image: linear-gradient(#1a73e8, #1a73e8);
  background-size: var(--progress) 100%;
  background-repeat: no-repeat;
  height: calc(0.25rem + 2px);
  margin: -1px 0;
}

/******** Firefox ********/
input[type="range"]::-moz-range-track {
  border-radius: 50rem;
  background-image: linear-gradient(#1a73e8, #1a73e8);
  background-size: var(--progress) 100%;
  background-repeat: no-repeat;
  height: calc(0.25rem + 2px);
  margin: -1px 0;
}

input[type="range"]::-webkit-slider-thumb {
  -webkit-appearance: none; /* Override default look */
  appearance: none;
  margin-top: -0.5rem; /* Centers thumb on the track */
  background-color: #1a73e8;
  height: 1.25rem;
  width: 1.25rem;
  border-radius: 50%;
}

input[type="range"]::-moz-range-thumb {
  border: none; /*Removes extra border that FF applies*/
  background-color: #1a73e8;
  height: 1.25rem;
  width: 1.25rem;
  border-radius: 50%;
}

input[type="range"]:focus {
  outline: none;
}

/***** Chrome, Safari, Opera, and Edge Chromium *****/
input[type="range"]:focus::-webkit-slider-thumb {
  box-shadow: 0 0 0 0.5rem rgba(13, 110, 253, 0.25);
}

/******** Firefox ********/
input[type="range"]:focus::-moz-range-thumb {
  box-shadow: 0 0 0 0.5rem rgba(13, 110, 253, 0.25);
}
</style>
