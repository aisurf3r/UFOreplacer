<script setup lang="ts">
import { ref, reactive, computed, watch, onMounted, onUnmounted, nextTick } from 'vue';
import hljs from 'highlight.js/lib/core';
import javascript from 'highlight.js/lib/languages/javascript';
import typescript from 'highlight.js/lib/languages/typescript';
import python from 'highlight.js/lib/languages/python';
import css from 'highlight.js/lib/languages/css';
import xml from 'highlight.js/lib/languages/xml';
import json from 'highlight.js/lib/languages/json';
import sql from 'highlight.js/lib/languages/sql';
import bash from 'highlight.js/lib/languages/bash';
import markdown from 'highlight.js/lib/languages/markdown';
import yaml from 'highlight.js/lib/languages/yaml';
import java from 'highlight.js/lib/languages/java';
import cpp from 'highlight.js/lib/languages/cpp';
import csharp from 'highlight.js/lib/languages/csharp';
import go from 'highlight.js/lib/languages/go';
import rust from 'highlight.js/lib/languages/rust';
import php from 'highlight.js/lib/languages/php';
import ruby from 'highlight.js/lib/languages/ruby';
import swift from 'highlight.js/lib/languages/swift';
import kotlin from 'highlight.js/lib/languages/kotlin';
import scala from 'highlight.js/lib/languages/scala';
import r from 'highlight.js/lib/languages/r';
import lua from 'highlight.js/lib/languages/lua';
import perl from 'highlight.js/lib/languages/perl';
import dockerfile from 'highlight.js/lib/languages/dockerfile';
import ini from 'highlight.js/lib/languages/ini';
import diff from 'highlight.js/lib/languages/diff';
import plaintext from 'highlight.js/lib/languages/plaintext';
import 'highlight.js/styles/atom-one-dark.css';

hljs.registerLanguage('javascript', javascript);
hljs.registerLanguage('typescript', typescript);
hljs.registerLanguage('python', python);
hljs.registerLanguage('css', css);
hljs.registerLanguage('xml', xml);
hljs.registerLanguage('json', json);
hljs.registerLanguage('sql', sql);
hljs.registerLanguage('bash', bash);
hljs.registerLanguage('markdown', markdown);
hljs.registerLanguage('yaml', yaml);
hljs.registerLanguage('java', java);
hljs.registerLanguage('cpp', cpp);
hljs.registerLanguage('csharp', csharp);
hljs.registerLanguage('go', go);
hljs.registerLanguage('rust', rust);
hljs.registerLanguage('php', php);
hljs.registerLanguage('ruby', ruby);
hljs.registerLanguage('swift', swift);
hljs.registerLanguage('kotlin', kotlin);
hljs.registerLanguage('scala', scala);
hljs.registerLanguage('r', r);
hljs.registerLanguage('lua', lua);
hljs.registerLanguage('perl', perl);
hljs.registerLanguage('dockerfile', dockerfile);
hljs.registerLanguage('ini', ini);
hljs.registerLanguage('diff', diff);
hljs.registerLanguage('plaintext', plaintext);

// ==================== INTERFACES ====================
interface Replacement {
  id: string;
  original: string;
  replacement: string;
  count: number;
  error?: string;
  color?: string;
}

interface AppState {
  replacements: Replacement[];
  options: {
    caseSensitive: boolean;
    wholeWord: boolean;
    useRegex: boolean;
  };
  textInput: string;
}

interface HistoryEntry {
  textInput?: string;
  replacements?: Partial<Replacement>[];
  options?: Partial<{
    caseSensitive: boolean;
    wholeWord: boolean;
    useRegex: boolean;
  }>;
}

interface HighlightMatch {
  element: HTMLElement;
  replacementId: string;
  matchIndex: number;
}

interface Translation {
  [key: string]: string;
}

interface Translations {
  en: Translation;
  es: Translation;
}

// ==================== CONSTANTES ====================
const HIGHLIGHT_COLORS = [
  '#FF6B6B', '#4ECDC4', '#FFD93D', '#6B5B95',
  '#45B7D1', '#FF9F40', '#88D8B0', '#EA5455',
  '#00743F', '#6A0572'
];

const MAX_HISTORY = 50;
const MAX_FILE_SIZE = 1024 * 150; 
const MAX_PRESET_NAME_LENGTH = 25;
const LONG_PRESS_DURATION = 600; 
const INPUT_DEBOUNCE_DELAY = 700; 

// ==================== TRADUCCIONES ====================
const translations: Translations = {
  en: {
    appTitle: "UFO Replacer",
    appSubtitle: " 🔎 Unidentified Finding & Overwriting mass tool",
    replacements: "Replacements",
    caseSensitive: "Case sensitive",
    wholeWord: "Whole words",
    useRegex: "Use regex",
    inputText: "Input Text",
    processedOutput: "Processed Output",
    add: "+ Add",
    clear: "Clear",
    undo: "↺",
    redo: "↻",
    presetName: "Preset name",
    save: "Save",
    load: "Load",
    delete: "Delete",
    copy: "Copy",
    download: "Download",
    processing: "Processing...",
    uploadFile: "Upload file",
    copied: "Copied to clipboard!",
    clearText: "Clear all text",
    matches: "{count} matches",
    tooltipUndo: "Undo (Ctrl+Z)",
    tooltipRedo: "Redo (Ctrl+Y)",
    replacePair: "Pair {number}",
    findPlaceholder: "Find...",
    replacePlaceholder: "Replace with...",
    inputPlaceholder: "Paste or upload text here...",
    outputPlaceholder: "Processed output will appear here...",
    fileError: "Unsupported file type",
    readError: "Error reading file",
    downloadError: "Error generating download",
    noTextError: "No text to process",
    invalidPattern: "Invalid pattern",
    replacementFailed: "Replacement failed",
    presetNameRequired: "Please enter a preset name",
    presetSaved: "Preset \"{name}\" saved!",
    presetLoaded: "Preset \"{name}\" loaded",
    presetDeleted: "Preset \"{name}\" deleted",
    presetNotFound: "Preset \"{name}\" not found",
    presetError: "Failed to {action} preset",
    fileSizeError: "File size exceeds 150kb limit",
    nextMatch: "Next match",
    prevMatch: "Previous match",
    codeMode: "Code",
    textMode: "Text"
  },
  es: {
    appTitle: "UFO Replacer",
    appSubtitle: " 🔎 Herramienta de Búsqueda y Reemplazo Masivo no identificada",
    replacements: "Reemplazos",
    caseSensitive: "Sensible a mayúsculas",
    wholeWord: "Palabras completas",
    useRegex: "Usar regex",
    inputText: "Texto de entrada",
    processedOutput: "Resultado procesado",
    add: "+ Añadir",
    clear: "Limpiar",
    undo: "↺",
    redo: "↻",
    presetName: "Nombre del preset",
    save: "Guardar",
    load: "Cargar",
    delete: "Eliminar",
    copy: "Copiar",
    download: "Descargar",
    processing: "Procesando...",
    uploadFile: "Subir archivo",
    copied: "¡Copiado al portapapeles!",
    clearText: "Limpiar todo el texto",
    matches: "{count} coincidencias",
    tooltipUndo: "Deshacer (Ctrl+Z)",
    tooltipRedo: "Rehacer (Ctrl+Y)",
    replacePair: "Par {number}",
    findPlaceholder: "Buscar...",
    replacePlaceholder: "Reemplazar con...",
    inputPlaceholder: "Pega o sube texto aquí...",
    outputPlaceholder: "El resultado aparecerá aquí...",
    fileError: "Tipo de archivo no soportado",
    readError: "Error leyendo archivo",
    downloadError: "Error generando descarga",
    noTextError: "No hay texto para procesar",
    invalidPattern: "Patrón inválido",
    replacementFailed: "Reemplazo fallido",
    presetNameRequired: "Ingrese un nombre para el preset",
    presetSaved: "¡Preset \"{name}\" guardado!",
    presetLoaded: "Preset \"{name}\" cargado",
    presetDeleted: "Preset \"{name}\" eliminado",
    presetNotFound: "Preset \"{name}\" no encontrado",
    presetError: "Error al {action} el preset",
    fileSizeError: "El archivo excede el límite de 150kb",
    nextMatch: "Siguiente coincidencia",
    prevMatch: "Coincidencia anterior",
    codeMode: "Código",
    textMode: "Texto"
  }
};

// ==================== ESTADO ====================
const EXT_TO_LANG: Record<string, string> = {
  '.js': 'javascript', '.mjs': 'javascript', '.cjs': 'javascript',
  '.ts': 'typescript', '.tsx': 'typescript',
  '.py': 'python', '.pyw': 'python',
  '.css': 'css', '.scss': 'css',
  '.html': 'xml', '.htm': 'xml', '.xml': 'xml', '.svg': 'xml',
  '.json': 'json',
  '.sql': 'sql',
  '.sh': 'bash', '.bash': 'bash', '.zsh': 'bash',
  '.md': 'markdown', '.markdown': 'markdown',
  '.yml': 'yaml', '.yaml': 'yaml',
  '.java': 'java',
  '.cpp': 'cpp', '.cc': 'cpp', '.cxx': 'cpp', '.hpp': 'cpp', '.h': 'cpp', '.c': 'cpp',
  '.cs': 'csharp',
  '.go': 'go',
  '.rs': 'rust',
  '.php': 'php',
  '.rb': 'ruby',
  '.swift': 'swift',
  '.kt': 'kotlin', '.kts': 'kotlin',
  '.scala': 'scala',
  '.r': 'r', '.R': 'r',
  '.lua': 'lua',
  '.pl': 'perl', '.pm': 'perl',
  '.csv': 'plaintext', '.txt': 'plaintext',
};

function detectLanguage(filename: string, content: string): string | undefined {
  const result = hljs.highlightAuto(content);
  if (result.relevance > 0 && result.language) return result.language;
  if (filename) {
    const ext = filename.substring(filename.lastIndexOf('.')).toLowerCase();
    const mapped = EXT_TO_LANG[ext];
    if (mapped && mapped !== 'plaintext') return mapped;
  }
  return undefined;
}

const history = ref<HistoryEntry[]>([]);
const historyIndex = ref(-1);
const replacements = ref<Replacement[]>([{
  id: crypto.randomUUID(),
  original: '',
  replacement: '',
  count: 0,
  color: HIGHLIGHT_COLORS[0]
}]);

const options = reactive({
  caseSensitive: false,
  wholeWord: false,
  useRegex: false
});

const textInput = ref('');
const fileContent = ref<string | null>(null);
const fileName = ref('');
const isLoading = ref(false);
const isProcessingLargeText = ref(false);
const errorMessage = ref('');
const inputContainerRef = ref<HTMLElement | null>(null);
const outputContainerRef = ref<HTMLElement | null>(null);
const presetName = ref('');
const activePanel = ref<'input'|'output'|'sidebar'>('input');
const lastFocusedReplacement = ref<number|null>(null);
const lastFocusedField = ref<'original'|'replacement'|null>(null);
const isProcessing = ref(false);
const regexCache = new Map<string, RegExp>();
const currentLang = ref<'en' | 'es'>('en');
const highlightMatchesData = ref<Record<string, HighlightMatch[]>>({});
const currentHighlightIndices = ref<Record<string, number>>({});
const isFirstNavigation = ref(true);
const isTyping = ref(false);
const longPressTimer = ref<number | null>(null);
const isLongPressing = ref(false);
const isPressing = ref(false);
const outputMode = ref<'code' | 'text'>('code');

// ==================== COMPUTED ====================
const canUndo = computed(() => historyIndex.value > 0);
const canRedo = computed(() => historyIndex.value < history.value.length - 1);
const processedText = computed(() => applyReplacements(textInput.value));

const canNavigateNext = computed(() => (id: string) => {
  const matches = highlightMatchesData.value[id] || [];
  return matches.length > 0 && (currentHighlightIndices.value[id] ?? 0) < matches.length - 1;
});

const canNavigatePrev = computed(() => (id: string) => {
  const matches = highlightMatchesData.value[id] || [];
  return matches.length > 0 && (currentHighlightIndices.value[id] ?? 0) > 0;
});

// ==================== FUNCIONES DE TRADUCCIÓN ====================
const t = (key: string, params?: Record<string, string|number>): string => {
  let text = translations[currentLang.value][key] || key;
  if (params) {
    Object.entries(params).forEach(([k, v]) => {
      text = text.replace(new RegExp(`{${k}}`, 'g'), v.toString());
    });
  }
  return text;
};

const setLanguage = (lang: 'en' | 'es') => {
  currentLang.value = lang;
  localStorage.setItem('ufoReplacerLang', lang);
};

// ==================== FUNCIONES DE HISTORIAL ====================
function saveState() {
  const newEntry: HistoryEntry = {};
  
  if (historyIndex.value >= 0) {
    const previous = history.value[historyIndex.value];

    if (previous.textInput !== textInput.value) {
      newEntry.textInput = textInput.value;
    }

    if (previous.options?.caseSensitive !== options.caseSensitive ||
        previous.options?.wholeWord !== options.wholeWord ||
        previous.options?.useRegex !== options.useRegex) {
      newEntry.options = { ...options };
    }

    const prevReplacements = previous.replacements || [];
    if (prevReplacements.length !== replacements.value.length) {
      newEntry.replacements = JSON.parse(JSON.stringify(replacements.value));
    } else {
      const changed = replacements.value.some((item, idx) => {
        const prev = prevReplacements[idx];
        return !prev || item.original !== prev.original || item.replacement !== prev.replacement;
      });
      
      if (changed) {
        newEntry.replacements = JSON.parse(JSON.stringify(replacements.value));
      }
    }
  } else {
    newEntry.textInput = textInput.value;
    newEntry.options = { ...options };
    newEntry.replacements = JSON.parse(JSON.stringify(replacements.value));
  }

  if (Object.keys(newEntry).length === 0) return;

  if (historyIndex.value < history.value.length - 1) {
    history.value = history.value.slice(0, historyIndex.value + 1);
  }

  history.value.push(newEntry);
  
  if (history.value.length > MAX_HISTORY) {
    history.value.shift();
  } else {
    historyIndex.value++;
  }
}

function undo() {
  if (canUndo.value) {
    historyIndex.value--;
    loadState(historyIndex.value);
  }
}

function redo() {
  if (canRedo.value) {
    historyIndex.value++;
    loadState(historyIndex.value);
  }
}

function loadState(index: number) {
  const newState: AppState = {
    replacements: [{
      id: crypto.randomUUID(),
      original: '',
      replacement: '',
      count: 0,
      color: HIGHLIGHT_COLORS[0]
    }],
    options: {
      caseSensitive: false,
      wholeWord: false,
      useRegex: false
    },
    textInput: ''
  };
  
  for (let i = 0; i <= index; i++) {
    const entry = history.value[i];
    
    if (entry.textInput !== undefined) {
      newState.textInput = entry.textInput;
    }
    
    if (entry.options) {
      Object.assign(newState.options, entry.options);
    }
    
    if (entry.replacements) {
      newState.replacements = JSON.parse(JSON.stringify(entry.replacements));
    }
  }
  
  replacements.value = newState.replacements;
  Object.assign(options, newState.options);
  textInput.value = newState.textInput;
  
  nextTick(() => {
    updateInputContent(newState.textInput);
    updateReplacementCounts(newState.textInput);
    processAndHighlight();
    forcePlaceholder();
  });
}

// ==================== FUNCIONES DE PRESETS ====================
function managePreset(action: 'save' | 'load' | 'delete', presetNameArg?: string) {
  const name = presetNameArg || presetName.value.trim();
  
  if (action === 'save') {
    if (!name) {
      showTempMessage('presetNameRequired', {}, 3000);
      return false;
    }
  }
  
  try {
    const presets = JSON.parse(localStorage.getItem('ufoReplacerPresets') || '{}');
    
    if (action === 'save') {
      presets[name] = {
        replacements: replacements.value,
        options: options
      };
      localStorage.setItem('ufoReplacerPresets', JSON.stringify(presets));
      showTempMessage('presetSaved', { name }, 3000);
      presetName.value = '';
    } 
    else if (action === 'load') {
      const preset = presets[name];
      if (preset) {
        replacements.value = preset.replacements.map((r: Replacement, idx: number) => ({
          ...r,
          id: r.id || crypto.randomUUID(),
          color: HIGHLIGHT_COLORS[idx % HIGHLIGHT_COLORS.length]
        }));
        Object.assign(options, preset.options);
        showTempMessage('presetLoaded', { name }, 3000);
        saveState();
        nextTick(() => {
          updateInputContent(textInput.value);
          updateReplacementCounts(textInput.value);
          processAndHighlight();
          forcePlaceholder();
        });
        return true;
      }
      showTempMessage('presetNotFound', { name }, 3000);
    } 
    else if (action === 'delete') {
      delete presets[name];
      localStorage.setItem('ufoReplacerPresets', JSON.stringify(presets));
      showTempMessage('presetDeleted', { name }, 3000);
    }
  } catch (error) {
    showTempMessage('presetError', { action }, 3000);
    console.error(`Preset ${action} error:`, error);
  }
  return false;
}

function savePreset() {
  managePreset('save');
}

function loadPreset(name: string) {
  return managePreset('load', name);
}

function deletePreset(name: string) {
  managePreset('delete', name);
}

function getPresetNames(): string[] {
  try {
    const presets = JSON.parse(localStorage.getItem('ufoReplacerPresets') || '{}');
    return Object.keys(presets).sort();
  } catch (error) {
    console.error('Error getting presets:', error);
    return [];
  }
}

// ==================== FUNCIONES DE REPLACEMENTS ====================
function addReplacement() {
  const newReplacement: Replacement = {
    id: crypto.randomUUID(),
    original: '',
    replacement: '',
    count: 0,
    color: HIGHLIGHT_COLORS[replacements.value.length % HIGHLIGHT_COLORS.length]
  };
  replacements.value.push(newReplacement);
  saveState();
  highlightMatchesData.value[newReplacement.id] = [];
  currentHighlightIndices.value[newReplacement.id] = 0;
  nextTick(() => {
    processAndHighlight();
  });
}

function removeReplacement(index: number) {
  if (replacements.value.length > 1) {
    const removedId = replacements.value[index].id;
    replacements.value.splice(index, 1);
    replacements.value.forEach((rep, idx) => {
      rep.color = HIGHLIGHT_COLORS[idx % HIGHLIGHT_COLORS.length];
    });
    delete highlightMatchesData.value[removedId];
    delete currentHighlightIndices.value[removedId];
    saveState();
    processAndHighlight();
  }
}

function clearAllReplacements() {
  replacements.value = [{
    id: crypto.randomUUID(),
    original: '',
    replacement: '',
    count: 0,
    color: HIGHLIGHT_COLORS[0]
  }];
  saveState();
  highlightMatchesData.value = {};
  currentHighlightIndices.value = {};
  clearHighlights();
  if (inputContainerRef.value) {
    inputContainerRef.value.blur();
  }
}

// ==================== FUNCIONES DE TEXTO ====================
function escapeRegExp(string: string): string {
  return string.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
}

function createSearchPattern(searchText: string): RegExp | null {
  if (!searchText) return null;

  try {
    const cacheKey = `${searchText}|${options.caseSensitive}|${options.wholeWord}|${options.useRegex}`;
    
    if (regexCache.has(cacheKey)) {
      return regexCache.get(cacheKey)!;
    }
    
    const flags = options.caseSensitive ? 'g' : 'gi';
    let pattern: string;

    if (options.useRegex) {
      const regex = new RegExp(searchText, flags);
      regexCache.set(cacheKey, regex);
      return regex;
    } else if (options.wholeWord) {
      pattern = `\\b${escapeRegExp(searchText)}\\b`;
    } else {
      pattern = escapeRegExp(searchText);
    }

    const regex = new RegExp(pattern, flags);
    regexCache.set(cacheKey, regex);
    return regex;
  } catch (error) {
    console.error('Error creating regex pattern:', error);
    return null;
  }
}

function updateReplacementCounts(text: string) {
  if (!text) {
    replacements.value.forEach(item => item.count = 0);
    return;
  }

  replacements.value.forEach(item => {
    item.error = undefined;
    if (!item.original) {
      item.count = 0;
      return;
    }

    const pattern = createSearchPattern(item.original);
    if (!pattern) {
      item.count = 0;
      return;
    }

    try {
      const matches = text.match(pattern);
      item.count = matches ? matches.length : 0;
    } catch (error) {
      console.error('Error counting matches:', error);
      item.count = 0;
      item.error = t('invalidPattern');
    }
  });
}

function applyReplacements(text: string): string {
  if (!text) return '';

  let result = text;
  
  replacements.value.forEach(item => {
    if (!item.original || item.error) return;

    const pattern = createSearchPattern(item.original);
    if (!pattern) return;

    try {
      const newText = result.replace(pattern, item.replacement);
      if (newText !== result) {
        result = newText;
        updateReplacementCounts(result);
      }
    } catch (error) {
      console.error('Error applying replacement:', error);
      item.error = t('replacementFailed');
    }
  });
  
  return result;
}

// ==================== FUNCIONES DE UI ====================
function clearHighlights() {
  if (!inputContainerRef.value) return;
  
  const content = inputContainerRef.value.innerText;
  inputContainerRef.value.innerHTML = '';
  inputContainerRef.value.textContent = content;
}

function processHighlightMatches() {
  if (!inputContainerRef.value || isTyping.value) return;

  const wasFocused = document.activeElement === inputContainerRef.value;
  const selection = window.getSelection();
  const range = selection?.rangeCount ? selection.getRangeAt(0) : null;
  const scrollTop = inputContainerRef.value.scrollTop;
  const plainText = inputContainerRef.value.innerText;

  inputContainerRef.value.innerHTML = '';
  inputContainerRef.value.appendChild(document.createTextNode(plainText));

  const tempContainer = document.createElement('div');
  tempContainer.textContent = plainText;
  
  const sortedReplacements = [...replacements.value]
    .filter(r => r.original && !r.error)
    .sort((a, b) => (b.original?.length || 0) - (a.original?.length || 0));

  highlightMatchesData.value = {};
  currentHighlightIndices.value = {};
  
  for (const item of sortedReplacements) {
    if (!item.original || item.error || !item.color) continue;
    
    const pattern = createSearchPattern(item.original);
    if (!pattern) continue;
    
    highlightMatchesData.value[item.id] = [];
    currentHighlightIndices.value[item.id] = 0;
    
    try {
      const walker = document.createTreeWalker(
        tempContainer,
        NodeFilter.SHOW_TEXT,
        null
      );
      
      const nodesToReplace: Text[] = [];
      let currentNode;
      
      while (currentNode = walker.nextNode()) {
        const textNode = currentNode as Text;
        const text = textNode.nodeValue || '';
        pattern.lastIndex = 0;
        if (pattern.test(text)) {
          nodesToReplace.push(textNode);
        }
      }
      
      for (const textNode of nodesToReplace) {
        const text = textNode.nodeValue || '';
        pattern.lastIndex = 0;
        
        const matches = [...text.matchAll(pattern)];
        if (matches.length === 0) continue;
        
        const nodeFragment = document.createDocumentFragment();
        let lastIndex = 0;
        
        matches.forEach((match, matchIdx) => {
          if (match.index === undefined) return;
          
          if (match.index > lastIndex) {
            nodeFragment.appendChild(
              document.createTextNode(text.slice(lastIndex, match.index))
            );
          }
          
          const span = document.createElement('span');
          span.className = 'highlight';
          span.style.backgroundColor = item.color!;
          span.textContent = match[0];
          span.dataset.replacementId = item.id;
          span.dataset.matchIndex = matchIdx.toString();
          nodeFragment.appendChild(span);

          highlightMatchesData.value[item.id].push({
            element: span,
            replacementId: item.id,
            matchIndex: matchIdx
          });
          
          lastIndex = match.index + match[0].length;
        });
        
        if (lastIndex < text.length) {
          nodeFragment.appendChild(
            document.createTextNode(text.slice(lastIndex))
          );
        }
        
        const parent = textNode.parentNode;
        if (parent) {
          parent.replaceChild(nodeFragment, textNode);
        }
      }
      
      if (highlightMatchesData.value[item.id].length > 0) {
        navigateToHighlight(item.id, 0, false);
      }
    } catch (error) {
      console.error('Error highlighting matches:', error);
    }
  }
  
  inputContainerRef.value.innerHTML = '';
  inputContainerRef.value.appendChild(tempContainer);

  if (wasFocused && selection && range) {
    inputContainerRef.value.focus();
    restoreCursorPosition(range.startOffset);
  }

  inputContainerRef.value.scrollTop = scrollTop;
}

function navigateToHighlight(replacementId: string, matchIndex: number, scroll: boolean = true, isLongPress: boolean = false) {
  const matches = highlightMatchesData.value[replacementId] || [];
  if (matches.length === 0 || matchIndex < 0 || matchIndex >= matches.length) {
    return;
  }

  currentHighlightIndices.value = {
    ...currentHighlightIndices.value,
    [replacementId]: matchIndex
  };

  nextTick(() => {
    nextTick(() => {
      matches.forEach(match => match.element.classList.remove('active'));

      const targetMatch = matches[matchIndex];
      if (targetMatch && targetMatch.element.isConnected) {
        targetMatch.element.classList.add('active');

        if (scroll && inputContainerRef.value) {
          void inputContainerRef.value.offsetHeight;

          if (isFirstNavigation.value) {
            targetMatch.element.scrollIntoView({
              behavior: 'smooth',
              block: 'center'
            });
            isFirstNavigation.value = false;
          } else {
            const containerRect = inputContainerRef.value.getBoundingClientRect();
            const elementRect = targetMatch.element.getBoundingClientRect();
            
            if (elementRect.top < containerRect.top || elementRect.bottom > containerRect.bottom) {
              targetMatch.element.scrollIntoView({
                behavior: 'smooth',
                block: 'center'
              });
            }
          }

          setTimeout(() => {
            const containerRect = inputContainerRef.value!.getBoundingClientRect();
            const elementRect = targetMatch.element.getBoundingClientRect();
            if (elementRect.top < containerRect.top || elementRect.bottom > containerRect.bottom) {
              const scrollTop = inputContainerRef.value!.scrollTop;
              const targetTop = elementRect.top - containerRect.top + scrollTop - (containerRect.height / 2) + (elementRect.height / 2);
              inputContainerRef.value!.scrollTo({
                top: targetTop,
                behavior: 'smooth'
              });
            }
          }, 100);
        }

        if (inputContainerRef.value && !isLongPress) {
          activePanel.value = 'input';
        }
      }
    });
  });
}

function navigateToNextHighlight(replacementId: string) {
  const matches = highlightMatchesData.value[replacementId] || [];
  if (matches.length === 0) return;

  let currentIndex = currentHighlightIndices.value[replacementId] ?? 0;
  const nextIndex = currentIndex + 1 < matches.length ? currentIndex + 1 : 0;

  navigateToHighlight(replacementId, nextIndex);
}

function navigateToPrevHighlight(replacementId: string) {
  const matches = highlightMatchesData.value[replacementId] || [];
  if (matches.length === 0) return; // Corrección: eliminamos IdleDeadline y ajustamos la condición

  let currentIndex = currentHighlightIndices.value[replacementId] ?? 0;
  const prevIndex = currentIndex > 0 ? currentIndex - 1 : matches.length - 1;

  navigateToHighlight(replacementId, prevIndex);
}

function jumpToFirstHighlight(replacementId: string) {
  navigateToHighlight(replacementId, 0, true, true);
}

function jumpToLastHighlight(replacementId: string) {
  const matches = highlightMatchesData.value[replacementId] || [];
  if (matches.length === 0) return;

  navigateToHighlight(replacementId, matches.length - 1, true, true);
}

function startLongPress(direction: 'prev' | 'next', replacementId: string) {
  if (isPressing.value) return;

  isLongPressing.value = false;
  isPressing.value = true;

  longPressTimer.value = window.setTimeout(() => {
    isLongPressing.value = true;
    if (direction === 'prev') {
      jumpToFirstHighlight(replacementId);
    } else {
      jumpToLastHighlight(replacementId);
    }
    isPressing.value = false;
  }, LONG_PRESS_DURATION);
}

function stopLongPress(direction: 'prev' | 'next', replacementId: string) {
  if (longPressTimer.value !== null) {
    clearTimeout(longPressTimer.value);
    longPressTimer.value = null;
  }

  if (isPressing.value && !isLongPressing.value) {
    if (direction === 'prev') {
      navigateToPrevHighlight(replacementId);
    } else {
      navigateToNextHighlight(replacementId);
    }
  }

  isLongPressing.value = false;
  isPressing.value = false;
}

function restoreCursorPosition(originalOffset: number) {
  if (!inputContainerRef.value || isTyping.value) return;
  
  const selection = window.getSelection();
  if (!selection) return;
  
  const textNodes = getTextNodes(inputContainerRef.value);
  let charCount = 0;
  let targetNode = null;
  let targetOffset = 0;
  
  for (const node of textNodes) {
    const length = node.nodeValue?.length || 0;
    
    if (charCount + length >= originalOffset) {
      targetNode = node;
      targetOffset = originalOffset - charCount;
      break;
    }
    
    charCount += length;
  }
  
  if (targetNode) {
    try {
      const newRange = document.createRange();
      const safeOffset = Math.min(targetOffset, targetNode.nodeValue?.length || 0);
      newRange.setStart(targetNode, safeOffset);
      newRange.collapse(true);
      
      selection.removeAllRanges();
      selection.addRange(newRange);
    } catch (e) {
      console.error('Error restoring cursor position:', e);
    }
  }
}

function getTextNodes(element: HTMLElement): Text[] {
  const walker = document.createTreeWalker(
    element,
    NodeFilter.SHOW_TEXT,
    null
  );
  const nodes: Text[] = [];
  let node;
  while (node = walker.nextNode()) {
    nodes.push(node as Text);
  }
  return nodes;
}

function handleInput() {
  if (!inputContainerRef.value) return;

  inputContainerRef.value.removeAttribute('data-empty');
  isTyping.value = true;
  const newText = inputContainerRef.value.innerText;

  if (textInput.value !== newText) {
    textInput.value = newText;
    saveState();
  }

  debouncedTyping();
}

function handlePaste(e: ClipboardEvent) {
  e.preventDefault();
  isTyping.value = false;
  const text = e.clipboardData?.getData('text/plain') || '';
  const selection = window.getSelection();
  
  if (selection && selection.rangeCount > 0 && inputContainerRef.value) {
    const range = selection.getRangeAt(0);
    range.deleteContents();
    
    const textNode = document.createTextNode(text);
    range.insertNode(textNode);
    
    range.setStartAfter(textNode);
    range.collapse(true);
    
    selection.removeAllRanges();
    selection.addRange(range);
    
    textInput.value = inputContainerRef.value.innerText;
    saveState();
  }
}

function copyToClipboard() {
  if (!processedText.value) return;
  
  navigator.clipboard.writeText(processedText.value)
    .then(() => {
      showTempMessage('copied');
    })
    .catch(err => {
      console.error('Failed to copy text: ', err);
      showTempMessage('copyError');
    });
}

function showTempMessage(messageKey: string, params?: Record<string, string|number>, duration = 2000) {
  errorMessage.value = t(messageKey, params);
  setTimeout(() => errorMessage.value = '', duration);
}

function forcePlaceholder() {
  if (!inputContainerRef.value) return;
  const isEmpty = !inputContainerRef.value.innerText || inputContainerRef.value.innerText.trim() === '';
  if (isEmpty) {
    inputContainerRef.value.innerHTML = '';
    inputContainerRef.value.setAttribute('data-empty', 'true');
  } else {
    inputContainerRef.value.removeAttribute('data-empty');
  }
}

function forceOutputPlaceholder() {
  if (!outputContainerRef.value) return;
  const isEmpty = !outputContainerRef.value.innerText || outputContainerRef.value.innerText.trim() === '';
  if (isEmpty) {
    outputContainerRef.value.innerHTML = '';
    outputContainerRef.value.setAttribute('data-empty', 'true');
  } else {
    outputContainerRef.value.removeAttribute('data-empty');
  }
}

function clearAllText() {
  textInput.value = '';
  fileName.value = '';
  fileContent.value = null;
  if (inputContainerRef.value) {
    inputContainerRef.value.innerHTML = '';
  }
  if (outputContainerRef.value) {
    outputContainerRef.value.innerHTML = '';
  }
  saveState();
  nextTick(() => {
    forcePlaceholder();
    forceOutputPlaceholder();
    processAndHighlight();
  });
}

function updateInputContent(content: string) {
  if (!inputContainerRef.value) return;
  
  inputContainerRef.value.innerHTML = '';
  if (content) {
    inputContainerRef.value.textContent = content;
  }
  if (!content && inputContainerRef.value.childNodes.length > 0) {
    inputContainerRef.value.innerHTML = '';
  }
  forcePlaceholder();
}

// ==================== FUNCIONES DE ARCHIVOS ====================
async function handleFileUpload(event: Event) {
  const file = (event.target as HTMLInputElement).files?.[0];
  if (!file) return;

  if (file.size > MAX_FILE_SIZE) {
    errorMessage.value = t('fileSizeError');
    (event.target as HTMLInputElement).value = '';
    return;
  }

  isLoading.value = true;
  errorMessage.value = '';
  fileName.value = file.name;

  const validTypes = [
    'text/plain', 'text/csv', 'text/html', 
    'text/css', 'application/json', 
    'application/javascript', 'text/xml',
    'application/xml', 'text/markdown'
  ];

  if (!validTypes.includes(file.type) && !file.name.match(/\.(txt|csv|html?|css|js|json|xml|md)$/i)) {
    errorMessage.value = t('fileError');
    isLoading.value = false;
    (event.target as HTMLInputElement).value = '';
    return;
  }

  try {
    const content = await readFileAsText(file);
    fileContent.value = content;
    isTyping.value = true;
    textInput.value = content;
    updateInputContent(content);
    saveState();
    await nextTick();
    isTyping.value = false;
    processAndHighlight();
  } catch (error) {
    errorMessage.value = t('readError');
    console.error('File read error:', error);
  } finally {
    isLoading.value = false;
  }
}

function readFileAsText(file: File): Promise<string> {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = e => resolve(e.target?.result as string);
    reader.onerror = () => reject(new Error('Failed to read file'));
    reader.readAsText(file);
  });
}

function processAndDownload() {
  if (!textInput.value) {
    errorMessage.value = t('noTextError');
    return;
  }

  try {
    const processed = processedText.value;
    const blob = new Blob([processed], { type: 'text/plain' });
    const url = URL.createObjectURL(blob);
    
    const a = document.createElement('a');
    a.href = url;
    a.download = fileName.value || 'processed.txt';
    document.body.appendChild(a);
    a.click();
    document.body.removeChild(a);
    URL.revokeObjectURL(url);
    errorMessage.value = '';
  } catch (error) {
    errorMessage.value = t('downloadError');
    console.error('Download error:', error);
  }
}

// ==================== FUNCIONES DE NAVEGACIÓN ====================
function focusPanel(panel: 'input' | 'output' | 'sidebar') {
  activePanel.value = panel;
  
  if (panel === 'input' && inputContainerRef.value) {
    inputContainerRef.value.focus();
  } 
  else if (panel === 'output' && outputContainerRef.value) {
    outputContainerRef.value.focus();
  } 
  else if (panel === 'sidebar') {
    const firstInput = document.querySelector('.replacement-item input') as HTMLElement;
    if (firstInput) {
      firstInput.focus();
      lastFocusedReplacement.value = 0;
      lastFocusedField.value = 'original';
    }
  }
}

function handleGlobalKeyDown(e: KeyboardEvent) {
  if (e.ctrlKey) {
    if (e.key === '1') {
      e.preventDefault();
      focusPanel('input');
    }
    else if (e.key === '2') {
      e.preventDefault();
      focusPanel('output');
    }
    else if (e.key === '3') {
      e.preventDefault();
      focusPanel('sidebar');
    }
    else if (e.key === 'z' && !e.shiftKey) {
      e.preventDefault();
      undo();
    }
    else if (e.key === 'y' || (e.shiftKey && e.key === 'Z')) {
      e.preventDefault();
      redo();
    }
  }
  
  if (activePanel.value === 'sidebar' && e.key === 'Tab') {
    e.preventDefault();
    navigateReplacements(e.shiftKey);
  }
}

function navigateReplacements(backward: boolean) {
  const currentIndex = lastFocusedReplacement.value ?? (backward ? 0 : -1);
  const newIndex = backward 
    ? Math.max(0, currentIndex - 1) 
    : currentIndex + 1;
    
  if (newIndex < replacements.value.length) {
    const inputs = document.querySelectorAll('.replacement-item input');
    if (inputs[newIndex * 2]) {
      (inputs[newIndex * 2] as HTMLElement).focus();
      lastFocusedReplacement.value = newIndex;
      lastFocusedField.value = 'original';
    }
  } else if (!backward) {
    addReplacement();
    nextTick(() => {
      const inputs = document.querySelectorAll('.replacement-item input');
      if (inputs[newIndex * 2]) {
        (inputs[newIndex * 2] as HTMLElement).focus();
        lastFocusedReplacement.value = newIndex;
        lastFocusedField.value = 'original';
      }
    });
  }
}

// ==================== FUNCIONES DE PROCESAMIENTO ====================
function debounce<T extends (...args: any[]) => any>(fn: T, delay: number): (...args: Parameters<T>) => void {
  let timer: number | null = null;
  return (...args: Parameters<T>) => {
    if (timer) clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay) as unknown as number;
  };
}

const debouncedProcessing = debounce(() => {
  if (!isProcessing.value) {
    processAndHighlight();
  }
}, 300);

const debouncedOriginalInput = debounce(() => {
  if (!isProcessing.value) {
    processAndHighlight();
  }
}, INPUT_DEBOUNCE_DELAY);

const debouncedTyping = debounce(() => {
  isTyping.value = false;
  if (!isProcessing.value) {
    processAndHighlight();
  }
}, INPUT_DEBOUNCE_DELAY);

function processAndHighlight() {
  isProcessing.value = true;
  isFirstNavigation.value = true;
  
  requestAnimationFrame(() => {
    try {
      clearRegexCache();
      updateReplacementCounts(textInput.value);
      processHighlightMatches();
      restoreFocusAfterUpdate();
      
      if (outputContainerRef.value) {
        outputContainerRef.value.innerHTML = '';
        outputContainerRef.value.removeAttribute('data-empty');
        if (processedText.value) {
          const wrapper = document.createElement('div');
          wrapper.className = 'output-content-wrapper';
          wrapper.setAttribute('tabindex', '-1');

          if (outputMode.value === 'code') {
            const codeElement = document.createElement('code');
            codeElement.setAttribute('tabindex', '-1');
            const lang = detectLanguage(fileName.value, processedText.value);
            if (lang) {
              codeElement.classList.add('language-' + lang);
            }
            try {
              const result = lang
                ? hljs.highlight(processedText.value, { language: lang })
                : hljs.highlightAuto(processedText.value);
              codeElement.innerHTML = result.value;
              if (result.language) {
                codeElement.classList.add('language-' + result.language);
              }
            } catch {
              codeElement.textContent = processedText.value;
              hljs.highlightElement(codeElement);
            }
            wrapper.appendChild(codeElement);
          } else {
            const textElement = document.createElement('div');
            textElement.className = 'output-text';
            textElement.setAttribute('tabindex', '-1');
            textElement.textContent = processedText.value;
            wrapper.appendChild(textElement);
          }

          outputContainerRef.value.appendChild(wrapper);
        } else {
          forceOutputPlaceholder();
        }
      }
    } catch (error) {
      console.error('Error during processing:', error);
    } finally {
      isProcessingLargeText.value = false;
      isProcessing.value = false;
    }
  });
}

function restoreFocusAfterUpdate() {
  if (lastFocusedReplacement.value !== null && lastFocusedField.value) {
    nextTick(() => {
      const inputs = document.querySelectorAll('.replacement-item input');
      const index = lastFocusedReplacement.value! * 2 + (lastFocusedField.value === 'replacement' ? 1 : 0);
      if (inputs[index]) {
        (inputs[index] as HTMLElement).focus();
      }
    });
  }
}

function clearRegexCache() {
  regexCache.clear();
}

// ==================== FUNCIONES DE FOCO ====================
function handleClickOutside(e: MouseEvent) {
  const target = e.target as HTMLElement;
  if (inputContainerRef.value && !inputContainerRef.value.contains(target)) {
    if (!target.closest('button, input, label, .form-check, .preset-section, .mode-switch')) {
      inputContainerRef.value.blur();
      forcePlaceholder();
    }
  }
}

// ==================== LIFECYCLE HOOKS ====================
onMounted(() => {
  const savedLang = localStorage.getItem('ufoReplacerLang') as 'en' | 'es';
  if (savedLang) currentLang.value = savedLang;

  if (inputContainerRef.value) {
    inputContainerRef.value.tabIndex = 0;
  }
  if (outputContainerRef.value) {
    outputContainerRef.value.tabIndex = 0;
  }
  document.addEventListener('keydown', handleGlobalKeyDown);
  document.addEventListener('click', handleClickOutside);
  saveState();
  nextTick(() => {
    forcePlaceholder();
    forceOutputPlaceholder();
  });
});

onUnmounted(() => {
  document.removeEventListener('keydown', handleGlobalKeyDown);
  document.removeEventListener('click', handleClickOutside);
  clearRegexCache();
});

// ==================== WATCHERS ====================
watch([textInput, () => [...replacements.value], () => ({ ...options }), outputMode], 
  ([newTextInput, newReplacements], [, oldReplacements]) => {
    const isLargeText = newTextInput.length > 10000;
    isProcessingLargeText.value = isLargeText;
    
    if (!isProcessing.value) {
      isProcessing.value = true;
      nextTick(() => {
        updateReplacementCounts(newTextInput);
        isProcessing.value = false;
      });
    }
    
    let originalChanged = false;
    if (newReplacements.length === oldReplacements.length) {
      originalChanged = newReplacements.some((item: Replacement, idx: number) => {
        const oldItem = oldReplacements[idx];
        return item.original !== oldItem.original && 
               item.replacement === oldItem.replacement && 
               item.id === oldItem.id;
      });
    }

    if (originalChanged) {
      debouncedOriginalInput();
    } else if (!isTyping.value) {
      debouncedProcessing();
    }
  }, 
  { deep: true }
);

watch(currentLang, () => {
  nextTick(() => {
    if (inputContainerRef.value) {
      inputContainerRef.value.setAttribute('data-placeholder', t('inputPlaceholder'));
      const isEmpty = !inputContainerRef.value.innerText || inputContainerRef.value.innerText.trim() === '';
      if (isEmpty) {
        inputContainerRef.value.setAttribute('data-empty', 'true');
      }
    }
    if (outputContainerRef.value) {
      outputContainerRef.value.setAttribute('data-placeholder', t('outputPlaceholder'));
      const isEmpty = !outputContainerRef.value.innerText || outputContainerRef.value.innerText.trim() === '';
      if (isEmpty) {
        outputContainerRef.value.setAttribute('data-empty', 'true');
      }
    }
  });
});
</script>

<template>
  <header class="header">
    <div class="header-content">
      <div class="header-titles">
        <h1 class="mb-0">{{ t('appTitle') }}</h1>
        <h6 class="subtitle-text">{{ t('appSubtitle') }}</h6>
      </div>
      <label class="lang-switch" :title="currentLang === 'en' ? 'Switch to Español' : 'Switch to English'">
        <input type="checkbox" :checked="currentLang === 'es'" @change="setLanguage(($event.target as HTMLInputElement).checked ? 'es' : 'en')">
        <span class="lang-slider">
          <span class="lang-option en">EN</span>
          <span class="lang-option es">ES</span>
        </span>
      </label>
    </div>
    <div v-if="errorMessage" class="alert alert-danger mt-2 mb-0 py-1">
      {{ errorMessage }}
    </div>
  </header>

  <main class="layout">
    <aside class="sidebar">
      <div class="sidebar-content">
        <div class="sidebar-header">
          <h5 class="card-title mb-0">{{ t('replacements') }}</h5>
          <div class="buttons">
            <button 
              @click="addReplacement" 
              class="btn btn-primary btn-sm"
              type="button"
            >
              {{ t('add') }}
            </button>
            <button 
              @click="clearAllReplacements" 
              class="btn btn-outline-danger btn-sm"
              type="button"
            >
              {{ t('clear') }}
            </button>
            <button
              @click="undo"
              class="btn btn-outline-secondary btn-sm"
              :disabled="!canUndo"
              :title="t('tooltipUndo')"
            >
              {{ t('undo') }}
            </button>
            <button
              @click="redo"
              class="btn btn-outline-secondary btn-sm"
              :disabled="!canRedo"
              :title="t('tooltipRedo')"
            >
              {{ t('redo') }}
            </button>
          </div>
        </div>

        <div class="options-section">
          <div class="form-check mb-2">
            <input 
              v-model="options.caseSensitive"
              class="form-check-input" 
              type="checkbox" 
              id="caseSensitive"
            >
            <label class="form-check-label" for="caseSensitive">
              {{ t('caseSensitive') }}
            </label>
          </div>
          <div class="form-check mb-2">
            <input 
              v-model="options.wholeWord"
              class="form-check-input" 
              type="checkbox" 
              id="wholeWord"
            >
            <label class="form-check-label" for="wholeWord">
              {{ t('wholeWord') }}
            </label>
          </div>
          <div class="form-check">
            <input 
              v-model="options.useRegex"
              class="form-check-input" 
              type="checkbox" 
              id="useRegex"
            >
            <label class="form-check-label" for="useRegex">
              {{ t('useRegex') }}
            </label>
          </div>
        </div>

        <div class="preset-section">
          <div class="input-group mb-2">
            <input
              v-model="presetName"
              type="text"
              class="form-control form-control-sm"
              :placeholder="t('presetName')"
              @keyup.enter="savePreset"
              :maxlength="MAX_PRESET_NAME_LENGTH"
            >
            <button
              @click="savePreset"
              class="btn btn-primary btn-sm"
              type="button"
            >
              {{ t('save') }}
            </button>
          </div>
          <div class="preset-list">
            <div v-for="preset in getPresetNames()" :key="preset" class="preset-item">
              <button
                @click="loadPreset(preset)"
                class="btn btn-sm btn-outline-secondary preset-btn"
              >
                {{ preset }}
              </button>
              <button
                @click.stop="deletePreset(preset)"
                class="btn btn-sm btn-outline-danger preset-delete"
                :title="t('delete')"
              >
                ×
              </button>
            </div>
          </div>
        </div>

        <div class="replacements-list">
          <div v-for="(item, index) in replacements" :key="item.id"
               class="replacement-item" :class="{ 'border-danger': item.error }">
            <div class="replacement-header">
              <small class="text-muted">{{ t('replacePair', { number: index + 1 }) }}</small>
              <div>
                <span v-if="item.count > 0" class="badge bg-primary me-2">
                  {{ t('matches', { count: item.count }) }}
                </span>
                <button
                  v-if="replacements.length > 1"
                  @click="removeReplacement(index)"
                  class="btn btn-danger btn-sm py-0 px-2"
                  type="button"
                  :aria-label="t('delete')"
                >
                  ×
                </button>
              </div>
            </div>
            <input
              v-model="item.original"
              type="text"
              class="form-control mb-2"
              :class="{ 'is-invalid': item.error }"
              :placeholder="t('findPlaceholder')"
              @focus="lastFocusedReplacement = index; lastFocusedField = 'original'"
              @change="saveState"
            >
            <input
              v-model="item.replacement"
              type="text"
              class="form-control"
              :placeholder="t('replacePlaceholder')"
              @focus="lastFocusedReplacement = index; lastFocusedField = 'replacement'"
              @change="saveState"
            >
            <div class="navigation-buttons mt-2">
              <button
                @mousedown="startLongPress('prev', item.id)"
                @mouseup="stopLongPress('prev', item.id)"
                @mouseleave="stopLongPress('prev', item.id)"
                @touchstart="startLongPress('prev', item.id)"
                @touchend="stopLongPress('prev', item.id)"
                class="btn btn-sm btn-outline-secondary"
                :disabled="!canNavigatePrev(item.id)"
                :title="t('prevMatch')"
              >
                ↑
              </button>
              <button
                @mousedown="startLongPress('next', item.id)"
                @mouseup="stopLongPress('next', item.id)"
                @mouseleave="stopLongPress('next', item.id)"
                @touchstart="startLongPress('next', item.id)"
                @touchend="stopLongPress('next', item.id)"
                class="btn btn-sm btn-outline-secondary ms-2"
                :disabled="!canNavigateNext(item.id)"
                :title="t('nextMatch')"
              >
                ↓
              </button>
            </div>
            <div v-if="item.error" class="invalid-feedback d-block">
              {{ item.error }}
            </div>
          </div>
        </div>

        <div class="sidebar-footer">
          <a href="https://github.com/aisurf3r/UFOreplacer" target="_blank" rel="noopener noreferrer" class="github-link">
            <svg class="github-icon" viewBox="0 0 16 16" fill="currentColor" width="18" height="18">
              <path d="M8 0C3.58 0 0 3.58 0 8c0 3.54 2.29 6.53 5.47 7.59.4.07.55-.17.55-.38 0-.19-.01-.82-.01-1.49-2.01.37-2.53-.49-2.69-.94-.09-.23-.48-.94-.82-1.13-.28-.15-.68-.52-.01-.53.63-.01 1.08.58 1.23.82.72 1.21 1.87.87 2.33.66.07-.52.28-.87.51-1.07-1.78-.2-3.64-.89-3.64-3.95 0-.87.31-1.59.82-2.15-.08-.2-.36-1.02.08-2.12 0 0 .67-.21 2.2.82.64-.18 1.32-.27 2-.27.68 0 1.36.09 2 .27 1.53-1.04 2.2-.82 2.2-.82.44 1.1.16 1.92.08 2.12.51.56.82 1.27.82 2.15 0 3.07-1.87 3.75-3.65 3.95.29.25.54.73.54 1.48 0 1.07-.01 1.93-.01 2.2 0 .21.15.46.55.38A8.013 8.013 0 0016 8c0-4.42-3.58-8-8-8z"/>
            </svg>
            <span>GitHub</span>
          </a>
        </div>
      </div>
    </aside>

    <section class="content">
      <div class="editor-panel">
        <div class="editor-header">
          <div class="input-header-content">
            <h5 class="mb-0">{{ t('inputText') }}</h5>
            <label class="mode-switch">
              <input type="checkbox" v-model="outputMode" true-value="code" false-value="text">
              <span class="slider">
                <span class="option code">{{ t('codeMode') }}</span>
                <span class="option text">{{ t('textMode') }}</span>
              </span>
            </label>
          </div>
          <div class="file-upload-container">
            <div class="file-formats">
              <span class="format-icon" title="Text">TXT</span>
              <span class="format-icon" title="CSV">CSV</span>
              <span class="format-icon" title="HTML">HTML</span>
              <span class="format-icon" title="CSS">CSS</span>
              <span class="format-icon" title="JavaScript">JS</span>
              <span class="format-icon" title="JSON">JSON</span>
              <span class="format-icon" title="XML">XML</span>
              <span class="format-icon" title="Markdown">MD</span>
            </div>
            <input 
              type="file" 
              class="form-control form-control-sm w-auto"
              @change="handleFileUpload"
              :aria-label="t('uploadFile')"
              accept=".txt,.csv,.html,.css,.js,.json,.xml,.md"
            >
          </div>
        </div>
        <div
          ref="inputContainerRef"
          class="editor-content"
          contenteditable="true"
          tabindex="0"
          :aria-label="t('inputText')"
          @focus="activePanel = 'input'"
          @blur="forcePlaceholder"
          @input="handleInput"
          @paste="handlePaste"
          :data-placeholder="t('inputPlaceholder')"
        ></div>
        <div v-if="isProcessingLargeText" class="loading-indicator">
          <div class="spinner-border spinner-border-sm"></div>
          <span>{{ t('processing') }}</span>
        </div>
      </div>

      <div class="editor-panel">
        <div class="editor-header">
          <h5 class="mb-0">{{ t('processedOutput') }}</h5>
          <div class="buttons">
            <button 
              @click="copyToClipboard" 
              class="btn btn-outline-secondary btn-sm"
              :disabled="!processedText"
            >
              {{ t('copy') }}
            </button>
            <button 
              @click="clearAllText" 
              class="btn btn-outline-danger btn-sm"
              :disabled="!textInput"
            >
              {{ t('clearText') }}
            </button>
            <button 
              @click="processAndDownload" 
              class="btn btn-primary btn-sm"
              :disabled="!processedText || isLoading"
            >
              <span v-if="isLoading" class="spinner-border spinner-border-sm me-1"></span>
              {{ isLoading ? t('processing') : t('download') }}
            </button>
          </div>
        </div>
        <pre 
          ref="outputContainerRef"
          class="editor-content output-pre"
          tabindex="0"
          :aria-label="t('processedOutput')"
          :data-placeholder="t('outputPlaceholder')"
        ></pre>
      </div>
    </section>
  </main>
</template>

<style>
:root {
  --sidebar-width: 350px;
  --header-height: 80px;
  --panel-header-height: 55px;
  --bg-color: #0a0e1a;
  --text-color: #e8ecf4;
  --sidebar-bg: rgba(15, 23, 42, 0.65);
  --panel-bg: rgba(15, 23, 42, 0.5);
  --border-color: rgba(148, 163, 184, 0.15);
  --input-bg: rgba(30, 41, 59, 0.6);
  --input-border: rgba(148, 163, 184, 0.2);
  --highlight-bg: rgba(30, 41, 59, 0.5);
  --card-bg: rgba(30, 41, 59, 0.45);
  --btn-hover-bg: rgba(51, 65, 85, 0.6);
  --glass-blur: 20px;
  --glass-border: 1px solid rgba(148, 163, 184, 0.12);
  --accent: #38bdf8;
  --accent-glow: rgba(56, 189, 248, 0.35);
}

body, html {
  margin: 0;
  padding: 0;
  height: 100%;
  width: 100%;
  overflow: hidden;
  background: var(--bg-color);
  color: var(--text-color);
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, sans-serif;
}

body::before {
  content: '';
  position: fixed;
  top: -50%;
  left: -50%;
  width: 200%;
  height: 200%;
  background:
    radial-gradient(ellipse at 20% 50%, rgba(56, 189, 248, 0.08) 0%, transparent 50%),
    radial-gradient(ellipse at 80% 20%, rgba(14, 165, 233, 0.06) 0%, transparent 50%),
    radial-gradient(ellipse at 60% 80%, rgba(6, 182, 212, 0.05) 0%, transparent 50%),
    radial-gradient(ellipse at 10% 90%, rgba(99, 102, 241, 0.04) 0%, transparent 40%);
  z-index: 0;
  pointer-events: none;
  animation: bgShift 20s ease-in-out infinite alternate;
}

@keyframes bgShift {
  0% { transform: translate(0, 0) rotate(0deg); }
  50% { transform: translate(-2%, 1%) rotate(1deg); }
  100% { transform: translate(1%, -1%) rotate(-0.5deg); }
}

.header {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: var(--header-height);
  background: var(--sidebar-bg);
  backdrop-filter: blur(var(--glass-blur));
  -webkit-backdrop-filter: blur(var(--glass-blur));
  padding: 10px 20px;
  border-bottom: var(--glass-border);
  z-index: 1000;
  display: flex;
  flex-direction: column;
  justify-content: center;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  gap: 20px;
  position: relative;
}

.header-titles {
  flex-grow: 1;
  min-width: 0;
}

.layout {
  display: flex;
  position: fixed;
  top: var(--header-height);
  left: 0;
  right: 0;
  bottom: 0;
  width: 100%;
  height: calc(100% - var(--header-height));
  z-index: 1;
}

.sidebar {
  width: var(--sidebar-width);
  height: 100%;
  border-right: var(--glass-border);
  overflow: hidden;
  flex-shrink: 0;
  background: var(--sidebar-bg);
  backdrop-filter: blur(var(--glass-blur));
  -webkit-backdrop-filter: blur(var(--glass-blur));
}

.sidebar-content {
  display: flex;
  flex-direction: column;
  height: 100%;
  padding: 15px;
  overflow: hidden;
}

.sidebar-footer {
  display: flex;
  justify-content: center;
  padding: 10px 0 4px 0;
  border-top: var(--glass-border);
  margin-top: 8px;
  flex-shrink: 0;
}

.github-link {
  display: flex;
  align-items: center;
  gap: 6px;
  color: rgba(148, 163, 184, 0.5);
  text-decoration: none;
  font-size: 0.75rem;
  transition: all 0.2s ease;
  padding: 4px 10px;
  border-radius: 8px;
}

.github-link:hover {
  color: var(--accent);
  background: rgba(56, 189, 248, 0.08);
}

.github-icon {
  flex-shrink: 0;
}

.sidebar-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.preset-section {
  margin-bottom: 15px;
  padding: 10px;
  border: var(--glass-border);
  border-radius: 12px;
  background: var(--card-bg);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
}

.preset-list {
  max-height: 120px;
  overflow-y: auto;
  margin-top: 10px;
  scrollbar-width: thin;
  scrollbar-color: rgba(148, 163, 184, 0.2) transparent;
}

.preset-list::-webkit-scrollbar {
  width: 6px;
}

.preset-list::-webkit-scrollbar-track {
  background: transparent;
}

.preset-list::-webkit-scrollbar-thumb {
  background-color: rgba(148, 163, 184, 0.2);
  border-radius: 3px;
}

.preset-item {
  display: flex;
  margin-bottom: 5px;
}

.preset-btn {
  flex-grow: 1;
  text-align: left;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  background: var(--input-bg);
  color: var(--text-color);
  border-color: var(--input-border);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  border-radius: 8px;
  transition: all 0.2s ease;
}

.preset-btn:hover {
  background: rgba(56, 189, 248, 0.1);
  border-color: rgba(56, 189, 248, 0.3);
}

.preset-delete {
  margin-left: 5px;
  padding: 0 5px;
  border-radius: 8px;
}

.replacements-list {
  flex-grow: 1;
  overflow-y: auto;
  overflow-x: hidden;
  margin-top: 15px;
  padding-right: 5px;
  scrollbar-width: thin;
  scrollbar-color: rgba(148, 163, 184, 0.2) transparent;
}

.replacements-list::-webkit-scrollbar {
  width: 6px;
}

.replacements-list::-webkit-scrollbar-track {
  background: transparent;
}

.replacements-list::-webkit-scrollbar-thumb {
  background-color: rgba(148, 163, 184, 0.2);
  border-radius: 3px;
}

.replacement-item {
  margin-bottom: 12px;
  padding: 12px;
  border: var(--glass-border);
  border-radius: 12px;
  background: var(--card-bg);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  transition: all 0.25s ease;
}

.replacement-item:hover {
  border-color: rgba(56, 189, 248, 0.2);
  box-shadow: 0 0 20px rgba(56, 189, 248, 0.05);
}

.replacement-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 8px;
}

.navigation-buttons {
  display: flex;
  justify-content: flex-end;
}

.options-section {
  padding: 15px;
  border: var(--glass-border);
  border-radius: 12px;
  margin-bottom: 15px;
  background: var(--card-bg);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
}

.content {
  flex-grow: 1;
  height: 100%;
  display: flex;
  flex-direction: column;
}

.editor-panel {
  display: flex;
  flex-direction: column;
  height: 50%;
  position: relative;
  background: var(--panel-bg);
  backdrop-filter: blur(var(--glass-blur));
  -webkit-backdrop-filter: blur(var(--glass-blur));
}

.editor-panel:first-child {
  border-bottom: var(--glass-border);
}

.editor-panel:focus-within {
  overflow: visible;
}

.editor-header {
  height: var(--panel-header-height);
  padding: 0 15px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  border-bottom: var(--glass-border);
  background: rgba(15, 23, 42, 0.3);
}

.input-header-content {
  display: flex;
  align-items: center;
  gap: 10px;
}

.mode-switch {
  position: relative;
  display: inline-block;
  width: 120px;
  height: 30px;
  margin-left: 10px;
}

.mode-switch input {
  opacity: 0;
  width: 0;
  height: 0;
}

.slider {
  position: absolute;
  cursor: pointer;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: var(--input-bg);
  border: var(--glass-border);
  border-radius: 15px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 5px;
  transition: all 0.3s ease;
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
}

.slider .option {
  font-size: 0.8rem;
  color: var(--text-color);
  opacity: 0.6;
  transition: opacity 0.3s ease;
  width: 50%;
  text-align: center;
  position: relative;
  z-index: 2;
}

.slider .option.code {
  text-align: left;
}

.slider .option.text {
  text-align: right;
}

.mode-switch input:checked + .slider .code {
  opacity: 1;
}

.mode-switch input:not(:checked) + .slider .text {
  opacity: 1;
}

.slider:before {
  position: absolute;
  content: "";
  height: 24px;
  width: 58px;
  border-radius: 12px;
  background: linear-gradient(135deg, #0ea5e9, #38bdf8);
  transition: transform 0.3s ease;
  top: 2px;
  left: 2px;
  z-index: 1;
  box-shadow: 0 0 10px rgba(56, 189, 248, 0.3);
}

.mode-switch input:checked + .slider:before {
  transform: translateX(0);
}

.mode-switch input:not(:checked) + .slider:before {
  transform: translateX(58px);
}

.editor-content {
  flex-grow: 1;
  overflow: auto;
  padding: 15px 10px;
  margin: 4px 4px 8px 4px;
  width: calc(100% - 8px);
  max-width: 100%;
  word-wrap: break-word;
  text-align: left;
  white-space: pre-wrap;
  background: transparent;
  color: var(--text-color);
  line-height: 1.5;
  box-sizing: border-box;
  position: relative;
  transition: all 0.3s ease;
  border-radius: 8px;
}

[contenteditable="true"] {
  outline: none;
  background: transparent;
  color: var(--text-color);
  text-align: left;
  font-family: 'SF Mono', 'Consolas', 'Monaco', monospace;
}

[contenteditable="true"]:empty:before,
[contenteditable="true"][data-empty="true"]:before {
  content: attr(data-placeholder);
  color: var(--text-color);
  opacity: 0.4;
  pointer-events: none;
  font-family: 'SF Mono', 'Consolas', 'Monaco', monospace;
  display: block;
}

[contenteditable="true"]:focus {
  outline: none;
  box-shadow: 0 0 0 2px rgba(56, 189, 248, 0.3), 0 0 20px rgba(56, 189, 248, 0.1);
  background: rgba(30, 41, 59, 0.3);
}

.output-pre {
  white-space: pre-wrap;
  word-wrap: break-word;
  background: rgba(30, 41, 59, 0.3);
  margin: 0;
  font-family: 'SF Mono', 'Consolas', 'Monaco', monospace;
  text-align: left;
  position: relative;
  padding: 15px 10px;
  margin: 4px 4px 8px 4px;
  width: calc(100% - 8px);
  max-width: 100%;
  box-sizing: border-box;
  overflow: auto;
  flex-grow: 1;
  transition: all 0.3s ease;
  border-radius: 8px;
}

.output-pre:empty:before,
.output-pre[data-empty="true"]:before {
  content: attr(data-placeholder);
  color: var(--text-color);
  opacity: 0.4;
  font-family: 'SF Mono', 'Consolas', 'Monaco', monospace;
  position: absolute;
  top: 15px;
  left: 10px;
  pointer-events: none;
}

.output-pre:focus {
  outline: none;
  box-shadow: 0 0 0 2px rgba(56, 189, 248, 0.3), 0 0 20px rgba(56, 189, 248, 0.1) !important;
  z-index: 1;
}

.output-content-wrapper {
  white-space: pre-wrap;
  word-wrap: break-word;
  font-family: 'SF Mono', 'Consolas', 'Monaco', monospace;
  color: var(--text-color);
  line-height: 1.5;
  width: 100%;
  height: 100%;
  transition: all 0.3s ease;
}

.output-content-wrapper code {
  white-space: pre-wrap !important;
  word-break: break-word !important;
  background: transparent !important;
  padding: 0 !important;
  margin: 0 !important;
  display: block;
  width: 100%;
  height: 100%;
  transition: all 0.3s ease;
}

.output-text {
  white-space: pre-wrap;
  word-wrap: break-word;
  font-family: 'SF Mono', 'Consolas', 'Monaco', monospace;
  color: var(--text-color);
  line-height: 1.5;
  padding: 0;
  margin: 0;
  width: 100%;
  height: 100%;
  transition: all 0.3s ease;
}

.buttons {
  display: flex;
  gap: 8px;
}

.btn-sm {
  font-size: 0.75rem;
  line-height: 1.2;
  padding: 0.25rem 0.5rem;
  border-radius: 8px;
  transition: all 0.2s ease;
}

.highlight {
  padding: 0 2px;
  border-radius: 3px;
  transition: all 0.15s ease;
}

.highlight.active {
  outline: 2px solid rgba(255, 255, 255, 0.9);
  outline-offset: 2px;
  z-index: 1;
  box-shadow: 0 0 8px rgba(255, 255, 255, 0.2);
}

.file-upload-container {
  display: flex;
  align-items: center;
  gap: 10px;
}

.file-formats {
  display: flex;
  gap: 5px;
}

.format-icon {
  font-size: 0.7rem;
  padding: 2px 5px;
  background: rgba(56, 189, 248, 0.1);
  border: 1px solid rgba(56, 189, 248, 0.15);
  border-radius: 4px;
  color: var(--accent);
  cursor: default;
  transition: all 0.2s ease;
}

.format-icon:hover {
  background: rgba(56, 189, 248, 0.15);
  border-color: rgba(56, 189, 248, 0.3);
}

.replacement-item input:focus {
  outline: none;
  box-shadow: 0 0 0 2px rgba(56, 189, 248, 0.3), 0 0 15px rgba(56, 189, 248, 0.1);
  border-color: rgba(56, 189, 248, 0.4);
}

.loading-indicator {
  position: absolute;
  bottom: 10px;
  right: 10px;
  background: rgba(15, 23, 42, 0.8);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  color: var(--accent);
  padding: 5px 10px;
  border-radius: 8px;
  border: var(--glass-border);
  font-size: 0.8rem;
  display: flex;
  align-items: center;
  gap: 5px;
}

.loading-indicator .spinner-border {
  width: 1rem;
  height: 1rem;
  border-width: 0.15em;
}

.form-control {
  background: var(--input-bg);
  color: var(--text-color);
  border: var(--glass-border);
  border-radius: 8px;
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  transition: all 0.2s ease;
}

.form-control:focus {
  background: rgba(30, 41, 59, 0.7);
  color: var(--text-color);
  border-color: rgba(56, 189, 248, 0.3);
  box-shadow: 0 0 15px rgba(56, 189, 248, 0.08);
}

.form-check-input {
  background-color: var(--input-bg);
  border-color: var(--input-border);
  transition: all 0.2s ease;
}

.form-check-input:checked {
  background-color: var(--accent);
  border-color: var(--accent);
  box-shadow: 0 0 8px rgba(56, 189, 248, 0.3);
}

.invalid-feedback {
  color: #f87171;
  font-size: 0.8rem;
  margin-top: 0.25rem;
}

.badge.bg-primary {
  background: linear-gradient(135deg, #0ea5e9, #38bdf8) !important;
  border: none;
  box-shadow: 0 0 10px rgba(56, 189, 248, 0.2);
}

.btn-outline-secondary {
  border-color: var(--input-border);
  color: var(--text-color);
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
  border-radius: 8px;
  transition: all 0.2s ease;
}

.btn-outline-secondary:hover {
  background: rgba(56, 189, 248, 0.1);
  border-color: rgba(56, 189, 248, 0.3);
  color: var(--accent);
}

.alert-danger {
  background: rgba(239, 68, 68, 0.85);
  backdrop-filter: blur(12px);
  -webkit-backdrop-filter: blur(12px);
  color: white;
  border: 1px solid rgba(239, 68, 68, 0.4);
  max-width: 300px;
  position: fixed;
  left: 10%;
  top: 15px;
  transform: translate(-50%, 0);
  z-index: 1050;
  padding: 0.5rem 1rem;
  border-radius: 8px;
  text-align: center;
  box-shadow: 0 8px 32px rgba(239, 68, 68, 0.2);
}

.form-control::placeholder,
.form-control-sm::placeholder {
  color: rgba(148, 163, 184, 0.5);
  opacity: 1;
}

.replacement-header small.text-muted {
  color: var(--accent) !important;
}

.lang-switch {
  position: relative;
  display: inline-block;
  width: 80px;
  height: 30px;
  margin-top: 8px;
  cursor: pointer;
}

.lang-switch input {
  opacity: 0;
  width: 0;
  height: 0;
  position: absolute;
}

.lang-slider {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: var(--input-bg);
  border: var(--glass-border);
  border-radius: 15px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 0 8px;
  transition: all 0.3s ease;
  backdrop-filter: blur(8px);
  -webkit-backdrop-filter: blur(8px);
}

.lang-slider .lang-option {
  font-size: 0.7rem;
  font-weight: 600;
  color: var(--text-color);
  opacity: 0.5;
  transition: opacity 0.3s ease;
  width: 50%;
  text-align: center;
  position: relative;
  z-index: 2;
  user-select: none;
}

.lang-slider .lang-option.en {
  text-align: left;
  padding-left: 2px;
}

.lang-slider .lang-option.es {
  text-align: right;
  padding-right: 2px;
}

.lang-switch input:not(:checked) + .lang-slider .en {
  opacity: 1;
}

.lang-switch input:checked + .lang-slider .es {
  opacity: 1;
}

.lang-slider:before {
  position: absolute;
  content: "";
  height: 24px;
  width: 38px;
  border-radius: 12px;
  background: linear-gradient(135deg, #0ea5e9, #38bdf8);
  transition: transform 0.3s ease;
  top: 2px;
  left: 2px;
  z-index: 1;
  box-shadow: 0 0 10px rgba(56, 189, 248, 0.3);
}

.lang-switch input:checked + .lang-slider:before {
  transform: translateX(38px);
}

@keyframes floatUFO {
  0%, 100% {
    transform: translateY(0);
  }
  50% {
    transform: translateY(-10px);
  }
}

h1 {
  display: inline-block;
  position: relative;
  margin: 0;
  background: linear-gradient(135deg, #e8ecf4, #38bdf8);
  -webkit-background-clip: text;
  -webkit-text-fill-color: transparent;
  background-clip: text;
}

h1::before {
  content: "🛸";
  display: inline-block;
  animation: floatUFO 3s ease-in-out infinite;
  margin-right: 8px;
  -webkit-text-fill-color: initial;
}

.subtitle-text {
  margin: 0;
  font-size: 0.9rem;
  opacity: 0.6;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
  color: rgba(148, 163, 184, 0.8);
}

.card-title {
  font-size: 1.1rem;
  font-weight: 500;
}

.input-group {
  display: flex;
  gap: 5px;
}

.input-group .form-control {
  flex-grow: 1;
}

.invalid-feedback.d-block {
  display: block;
}

.border-danger {
  border-color: rgba(239, 68, 68, 0.6) !important;
  box-shadow: 0 0 15px rgba(239, 68, 68, 0.1);
}

.btn-danger {
  background: linear-gradient(135deg, #ef4444, #dc2626);
  border-color: rgba(239, 68, 68, 0.4);
  border-radius: 8px;
  transition: all 0.2s ease;
}

.btn-danger:hover {
  background: linear-gradient(135deg, #dc2626, #b91c1c);
  border-color: rgba(220, 38, 38, 0.5);
  box-shadow: 0 0 15px rgba(239, 68, 68, 0.2);
}

.btn-primary {
  background: linear-gradient(135deg, #0ea5e9, #38bdf8);
  border-color: rgba(56, 189, 248, 0.4);
  border-radius: 8px;
  transition: all 0.2s ease;
}

.btn-primary:hover {
  background: linear-gradient(135deg, #0284c7, #0ea5e9);
  border-color: rgba(14, 165, 233, 0.5);
  box-shadow: 0 0 15px rgba(56, 189, 248, 0.2);
}

.btn-outline-danger {
  color: #f87171;
  border-color: rgba(239, 68, 68, 0.4);
  border-radius: 8px;
  transition: all 0.2s ease;
}

.btn-outline-danger:hover {
  background: rgba(239, 68, 68, 0.15);
  color: #f87171;
  border-color: rgba(239, 68, 68, 0.5);
}

.spinner-border {
  display: inline-block;
  width: 1rem;
  height: 1rem;
  vertical-align: text-bottom;
  border: 0.15em solid currentColor;
  border-right-color: transparent;
  border-radius: 50%;
  animation: spinner-border 0.75s linear infinite;
}

@keyframes spinner-border {
  to { transform: rotate(360deg); }
}

.me-1 {
  margin-right: 0.25rem;
}

.me-2 {
  margin-right: 0.5rem;
}

.mb-0 {
  margin-bottom: 0 !important;
}

.mb-2 {
  margin-bottom: 0.5rem !important;
}

.mt-2 {
  margin-top: 0.5rem !important;
}

.py-0 {
  padding-top: 0 !important;
  padding-bottom: 0 !important;
}

.py-1 {
  padding-top: 0.25rem !important;
  padding-bottom: 0.25rem !important;
}

.px-2 {
  padding-left: 0.5rem !important;
  padding-right: 0.5rem !important;
}

.d-block {
  display: block !important;
}

.w-auto {
  width: auto !important;
}

.text-muted {
  color: rgba(148, 163, 184, 0.6) !important;
}

.is-invalid {
  border-color: rgba(239, 68, 68, 0.6) !important;
}

.d-none {
  display: none !important;
}

.align-items-center {
  align-items: center !important;
}

.justify-content-between {
  justify-content: space-between !important;
}

.flex-grow-1 {
  flex-grow: 1 !important;
}

.position-relative {
  position: relative !important;
}

.text-center {
  text-align: center !important;
}

.overflow-hidden {
  overflow: hidden !important;
}

.whitespace-nowrap {
  white-space: nowrap !important;
}

.text-ellipsis {
  text-overflow: ellipsis !important;
}

.cursor-default {
  cursor: default !important;
}

.transition-all {
  transition: all 0.2s ease !important;
}

.h-fit-content {
  height: fit-content !important;
}

.scrollbar-thin::-webkit-scrollbar {
  width: 6px !important;
}

.scrollbar-thin::-webkit-scrollbar-track {
  background: transparent !important;
}

.scrollbar-thin::-webkit-scrollbar-thumb {
  background-color: rgba(148, 163, 184, 0.2) !important;
  border-radius: 3px !important;
}

.ms-2 {
  margin-left: 0.5rem !important;
}

.form-check {
  transition: all 0.2s ease;
}

.form-check:hover {
  transform: translateX(2px);
}

.form-check-label {
  transition: color 0.2s ease;
}

.form-check-label:hover {
  color: var(--accent);
}

button {
  transition: all 0.2s ease;
}

button:hover {
  transform: translateY(-1px);
}

button:active {
  transform: translateY(0);
}

.replacement-item input {
  transition: all 0.2s ease;
  border-radius: 8px;
}

@keyframes glassShimmer {
  0% { background-position: -200% 0; }
  100% { background-position: 200% 0; }
}

.editor-panel:focus-within .editor-header {
  background: rgba(56, 189, 248, 0.05);
  border-bottom-color: rgba(56, 189, 248, 0.15);
}
</style>