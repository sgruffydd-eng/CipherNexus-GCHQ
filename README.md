import React, { useEffect, useMemo, useRef, useState } from 'react';

const themeDefinitions = {
  gchq: {
    name: 'GCHQ Dark',
    bg: '#0b132b',
    panel: '#1c2541',
    text: '#e2e8f0',
    primary: '#4cc9f0',
    accent: '#7dd3fc',
    success: '#22c55e',
    font: 'sans'
  },
  matrix: {
    name: 'Matrix Terminal',
    bg: '#020b07',
    panel: '#0b1a10',
    text: '#d6ffe8',
    primary: '#00ff66',
    accent: '#7cffae',
    success: '#00ff66',
    font: 'mono'
  },
  parchment: {
    name: 'Parchment',
    bg: '#f4ead7',
    panel: '#e8d9c2',
    text: '#1d120d',
    primary: '#8a5c3b',
    accent: '#b57d52',
    success: '#2ca167',
    font: 'serif'
  },
  midnight: {
    name: 'Midnight Cyber',
    bg: '#0f172a',
    panel: '#111827',
    text: '#dbeafe',
    primary: '#22d3ee',
    accent: '#c084fc',
    success: '#34d399',
    font: 'sans'
  },
  academic: {
    name: 'Academic Light',
    bg: '#f5f7f9',
    panel: '#e2e8f0',
    text: '#0f172a',
    primary: '#2563eb',
    accent: '#1d4ed8',
    success: '#15803d',
    font: 'sans'
  }
} as const;

type ThemeKey = keyof typeof themeDefinitions;
type LayoutMode = 'split' | 'stacked';
type FontMode = 'sans' | 'mono' | 'serif';
type CipherGroup = 'Substitution' | 'Polyalphabetic' | 'Transposition' | 'Historical' | 'Modern';

type Algorithm = {
  label: string;
  group: CipherGroup;
  type: 'encrypt' | 'decrypt' | 'both';
};

const defaultTheme: ThemeKey = 'gchq';
const algorithmList: Algorithm[] = [
  { label: 'Caesar', group: 'Substitution', type: 'both' },
  { label: 'ROT13', group: 'Substitution', type: 'both' },
  { label: 'ROT47', group: 'Substitution', type: 'both' },
  { label: 'Atbash', group: 'Substitution', type: 'both' },
  { label: 'Affine', group: 'Substitution', type: 'both' },
  { label: 'Monoalphabetic', group: 'Substitution', type: 'both' },
  { label: 'Keyword', group: 'Substitution', type: 'both' },
  { label: 'Baconian', group: 'Substitution', type: 'both' },
  { label: 'Polybius', group: 'Substitution', type: 'both' },
  { label: 'Morse', group: 'Substitution', type: 'both' },
  { label: 'A1Z26', group: 'Substitution', type: 'both' },
  { label: 'Vigenère', group: 'Polyalphabetic', type: 'both' },
  { label: 'Beaufort', group: 'Polyalphabetic', type: 'both' },
  { label: 'Autokey', group: 'Polyalphabetic', type: 'both' },
  { label: 'Gronsfeld', group: 'Polyalphabetic', type: 'both' },
  { label: 'Trithemius', group: 'Polyalphabetic', type: 'both' },
  { label: 'Playfair', group: 'Polyalphabetic', type: 'both' },
  { label: 'Two-Square', group: 'Polyalphabetic', type: 'both' },
  { label: 'Four-Square', group: 'Polyalphabetic', type: 'both' },
  { label: 'Columnar', group: 'Transposition', type: 'both' },
  { label: 'Rail Fence', group: 'Transposition', type: 'both' },
  { label: 'Route', group: 'Transposition', type: 'both' },
  { label: 'Myszkowski', group: 'Transposition', type: 'both' },
  { label: 'Scytale', group: 'Transposition', type: 'both' },
  { label: 'Anagram', group: 'Transposition', type: 'both' },
  { label: 'Reverser', group: 'Transposition', type: 'both' },
  { label: 'Fleissner Grille', group: 'Transposition', type: 'both' },
  { label: 'Enigma', group: 'Historical', type: 'both' },
  { label: 'M-209', group: 'Historical', type: 'both' },
  { label: 'Jefferson Disk', group: 'Historical', type: 'both' },
  { label: 'Lorenz SZ40/42', group: 'Historical', type: 'both' },
  { label: 'Alberti Disk', group: 'Historical', type: 'both' },
  { label: 'Binary', group: 'Modern', type: 'both' },
  { label: 'Hex', group: 'Modern', type: 'both' },
  { label: 'Octal', group: 'Modern', type: 'both' },
  { label: 'Base64', group: 'Modern', type: 'both' },
  { label: 'Base32', group: 'Modern', type: 'both' },
  { label: 'XOR', group: 'Modern', type: 'both' },
  { label: 'Vernam', group: 'Modern', type: 'both' },
  { label: 'Tap Code', group: 'Modern', type: 'both' },
  { label: 'Bifid', group: 'Modern', type: 'both' },
  { label: 'Trifid', group: 'Modern', type: 'both' },
  { label: 'Book Cipher', group: 'Modern', type: 'both' }
];

const alphabet = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ';
const englishFrequencies: Record<string, number> = {
  A: 8.167, B: 1.492, C: 2.782, D: 4.253, E: 12.702, F: 2.228, G: 2.015, H: 6.094, I: 6.966,
  J: 0.153, K: 0.772, L: 4.025, M: 2.406, N: 6.749, O: 7.507, P: 1.929, Q: 0.095, R: 5.987,
  S: 6.327, T: 9.056, U: 2.758, V: 0.978, W: 2.360, X: 0.150, Y: 1.974, Z: 0.074
};

const commonWords = ['THE', 'AND', 'ING', 'TION', 'HER', 'THAT', 'FOR', 'NOT', 'WITH', 'THIS', 'HAVE', 'FROM', 'THEY'];

function normalizeText(text: string): string {
  return text.replace(/\r/g, '');
}

function toAlphabetLetters(text: string): string {
  return text.toUpperCase().replace(/[^A-Z]/g, '');
}

function shiftAlphabet(shift: number) {
  return alphabet.slice(shift) + alphabet.slice(0, shift);
}

function caesarTransform(input: string, shift: number, decrypt = false): string {
  const direction = decrypt ? -shift : shift;
  return input
    .split('')
    .map((char) => {
      if (/[A-Z]/.test(char)) {
        const index = alphabet.indexOf(char.toUpperCase());
        const shifted = (index + direction + 26) % 26;
        return alphabet[shifted];
      }
      if (/[a-z]/.test(char)) {
        const index = alphabet.indexOf(char.toUpperCase());
        const shifted = (index + direction + 26) % 26;
        return alphabet[shifted].toLowerCase();
      }
      return char;
    })
    .join('');
}

function rot13(text: string): string {
  return text.replace(/[A-Za-z]/g, (char) => {
    const base = char <= 'Z' ? 65 : 97;
    return String.fromCharCode(base + ((char.charCodeAt(0) - base + 13) % 26));
  });
}

function rot47(text: string): string {
  return text.replace(/./g, (char) => {
    const code = char.charCodeAt(0);
    if (code >= 33 && code <= 126) {
      const shifted = 33 + ((code - 33 + 47) % 94);
      return String.fromCharCode(shifted);
    }
    return char;
  });
}

function atbashTransform(input: string): string {
  return input
    .split('')
    .map((char) => {
      if (/[A-Z]/.test(char)) {
        const index = alphabet.indexOf(char.toUpperCase());
        return alphabet[25 - index];
      }
      if (/[a-z]/.test(char)) {
        const index = alphabet.indexOf(char.toUpperCase());
        return alphabet[25 - index].toLowerCase();
      }
      return char;
    })
    .join('');
}

function affineTransform(input: string, a: number, b: number, decrypt = false): string {
  const inverseMod = (value: number) => {
    for (let i = 1; i < 26; i += 1) {
      if ((value * i) % 26 === 1) return i;
    }
    return 1;
  };

  const modInverse = inverseMod(a);
  return input
    .split('')
    .map((char) => {
      if (/[A-Z]/.test(char)) {
        const index = alphabet.indexOf(char.toUpperCase());
        const transformed = decrypt ? (modInverse * ((index - b + 26) % 26)) % 26 : (a * index + b) % 26;
        return alphabet[transformed];
      }
      if (/[a-z]/.test(char)) {
        const index = alphabet.indexOf(char.toUpperCase());
        const transformed = decrypt ? (modInverse * ((index - b + 26) % 26)) % 26 : (a * index + b) % 26;
        return alphabet[transformed].toLowerCase();
      }
      return char;
    })
    .join('');
}

function keywordAlphabet(key: string): string {
  const build = (keyText: string) => {
    const seen = new Set<string>();
    const letters: string[] = [];
    for (const char of keyText.toUpperCase()) {
      if (/[A-Z]/.test(char) && !seen.has(char)) {
        seen.add(char);
        letters.push(char);
      }
    }
    for (const char of alphabet) {
      if (!seen.has(char)) letters.push(char);
    }
    return letters.join('');
  };
  return build(key);
}

function monoalphabeticTransform(input: string, key: string, decrypt = false): string {
  const mono = keywordAlphabet(key);
  const lookup = new Map<string, string>();
  for (let i = 0; i < alphabet.length; i += 1) {
    lookup.set(alphabet[i], mono[i]);
    lookup.set(alphabet[i].toLowerCase(), mono[i].toLowerCase());
  }

  return input
    .split('')
    .map((char) => {
      if (/[A-Z]/.test(char)) return decrypt ? alphabet[mono.indexOf(char)] : mono[alphabet.indexOf(char)];
      if (/[a-z]/.test(char)) return decrypt ? alphabet[mono.indexOf(char.toUpperCase())].toLowerCase() : mono[alphabet.indexOf(char.toUpperCase())].toLowerCase();
      return char;
    })
    .join('');
}

function vigenereTransform(input: string, key: string, decrypt = false): string {
  const sanitized = key.toUpperCase().replace(/[^A-Z]/g, '');
  if (!sanitized) return input;

  return input.split('').map((char, index) => {
    const keyChar = sanitized[index % sanitized.length];
    const keyShift = alphabet.indexOf(keyChar);

    if (/[A-Z]/.test(char)) {
      const plainIndex = alphabet.indexOf(char.toUpperCase());
      const resultIndex = decrypt ? (plainIndex - keyShift + 26) % 26 : (plainIndex + keyShift) % 26;
      return alphabet[resultIndex];
    }
    if (/[a-z]/.test(char)) {
      const plainIndex = alphabet.indexOf(char.toUpperCase());
      const resultIndex = decrypt ? (plainIndex - keyShift + 26) % 26 : (plainIndex + keyShift) % 26;
      return alphabet[resultIndex].toLowerCase();
    }
    return char;
  }).join('');
}

function beaufordTransform(input: string, key: string, decrypt = false): string {
  const sanitized = key.toUpperCase().replace(/[^A-Z]/g, '');
  if (!sanitized) return input;

  return input.split('').map((char, index) => {
    const keyChar = sanitized[index % sanitized.length];
    const keyShift = alphabet.indexOf(keyChar);
    if (/[A-Z]/.test(char)) {
      const plainIndex = alphabet.indexOf(char.toUpperCase());
      const resultIndex = decrypt ? (plainIndex + keyShift) % 26 : (plainIndex - keyShift + 26) % 26;
      return alphabet[resultIndex];
    }
    if (/[a-z]/.test(char)) {
      const plainIndex = alphabet.indexOf(char.toUpperCase());
      const resultIndex = decrypt ? (plainIndex + keyShift) % 26 : (plainIndex - keyShift + 26) % 26;
      return alphabet[resultIndex].toLowerCase();
    }
    return char;
  }).join('');
}

function railFenceTransform(text: string, rails: number, decrypt = false): string {
  if (rails <= 1 || !text) return text;
  const entries = Array.from({ length: rails }, () => [] as string[]);

  if (!decrypt) {
    let rail = 0;
    let direction = 1;
    for (const char of text) {
      entries[rail].push(char);
      if (rail === 0) direction = 1;
      if (rail === rails - 1) direction = -1;
      rail += direction;
    }
    return entries.map((railText) => railText.join('')).join('');
  }

  const railPattern: number[] = [];
  let rail = 0;
  let direction = 1;
  for (let i = 0; i < text.length; i += 1) {
    railPattern.push(rail);
    if (rail === 0) direction = 1;
    if (rail === rails - 1) direction = -1;
    rail += direction;
  }

  const railLengths = Array.from({ length: rails }, () => 0);
  for (const loc of railPattern) railLengths[loc] += 1;
  const segments: string[] = [];
  let cursor = 0;
  for (let i = 0; i < rails; i += 1) {
    segments.push(text.slice(cursor, cursor + railLengths[i]));
    cursor += railLengths[i];
  }
  const railChars: string[][] = Array.from({ length: rails }, () => []);
  let segmentIndex = 0;
  for (let i = 0; i < railPattern.length; i += 1) {
    railChars[railPattern[i]].push(segments[railPattern[i]][0]);
    segments[railPattern[i]] = segments[railPattern[i]].slice(1);
  }

  const out: string[] = Array(text.length).fill('');
  let idx = 0;
  for (let i = 0; i < railPattern.length; i += 1) {
    const rail = railPattern[i];
    out[i] = segments[rail][0];
  }
  return out.join('');
}

function railFenceCandidate(text: string): string[] {
  const results: string[] = [];
  for (let rails = 2; rails <= 7; rails += 1) {
    results.push(railFenceTransform(text, rails, false));
  }
  return results;
}

function binaryEncode(text: string): string {
  return text
    .split('')
    .map((char) => char.charCodeAt(0).toString(2).padStart(8, '0'))
    .join(' ');
}

function binaryDecode(text: string): string {
  return text
    .split(/\s+/)
    .filter(Boolean)
    .map((chunk) => String.fromCharCode(parseInt(chunk, 2)))
    .join('');
}

function hexEncode(text: string): string {
  return Array.from(text).map((char) => char.charCodeAt(0).toString(16).padStart(2, '0')).join(' ');
}

function hexDecode(text: string): string {
  const cleaned = text.replace(/\s+/g, '');
  return Buffer.from(cleaned, 'hex').toString('utf8');
}

function octalEncode(text: string): string {
  return Array.from(text).map((char) => char.charCodeAt(0).toString(8).padStart(3, '0')).join(' ');
}

function octalDecode(text: string): string {
  return text
    .split(/\s+/)
    .filter(Boolean)
    .map((chunk) => String.fromCharCode(parseInt(chunk, 8)))
    .join('');
}

function base64Encode(text: string): string {
  return btoa(unescape(encodeURIComponent(text)));
}

function base64Decode(text: string): string {
  return decodeURIComponent(escape(atob(text)));
}

function base32Encode(text: string): string {
  const bytes = new TextEncoder().encode(text);
  let bits = 0;
  let value = 0;
  let output = '';
  const alphabet32 = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ234567';

  for (let i = 0; i < bytes.length; i += 1) {
    value = (value << 8) | bytes[i];
    bits += 8;
    while (bits >= 5) {
      bits -= 5;
      output += alphabet32[(value >> bits) & 31];
    }
  }

  if (bits > 0) {
    output += alphabet32[(value << (5 - bits)) & 31];
  }

  return output.padEnd(Math.ceil(output.length / 8) * 8, '=');
}

function alphabetToMap(list: string[]) {
  const result = new Map<string, string>();
  list.forEach((token) => {
    if (token.length === 2) {
      result.set(token[0].toUpperCase(), token[1]);
    }
  });
  return result;
}

function morseEncode(text: string): string {
  const morseMap: Record<string, string> = {
    A: '.-', B: '-...', C: '-.-.', D: '-..', E: '.', F: '..-.', G: '--.', H: '....', I: '..', J: '.---',
    K: '-.-', L: '.-..', M: '--', N: '-.', O: '---', P: '.--.', Q: '--.-', R: '.-.', S: '...', T: '-', U: '..-',
    V: '...-', W: '.--', X: '-..-', Y: '-.--', Z: '--..', 0: '-----', 1: '.----', 2: '..---', 3: '...--',
    4: '....-', 5: '.....', 6: '-....', 7: '--...', 8: '---..', 9: '----.', ' ': '/', '.': '.-.-.-', ',': '--..--'
  };

  return Array.from(text.toUpperCase()).map((char) => morseMap[char] ?? char).join(' ');
}

function morseDecode(text: string): string {
  const morseMap = new Map<string, string>(Object.entries({
    '.-': 'A', '-...': 'B', '-.-.': 'C', '-..': 'D', '.': 'E', '..-.': 'F', '--.': 'G', '....': 'H', '..': 'I', '.---': 'J',
    '-.-': 'K', '.-..': 'L', '--': 'M', '-.': 'N', '---': 'O', '.--.': 'P', '--.-': 'Q', '.-.': 'R', '...': 'S', '-': 'T',
    '..-': 'U', '...-': 'V', '.--': 'W', '-..-': 'X', '-.--': 'Y', '--..': 'Z', '-----': '0', '.----': '1', '..---': '2',
    '...--': '3', '....-': '4', '.....': '5', '-....': '6', '--...': '7', '---..': '8', '----.': '9', '/': ' ' 
  }));

  return text.split(' ').map((token) => morseMap.get(token) ?? token).join('');
}

function a1z26Encode(text: string): string {
  return Array.from(text.toUpperCase()).filter((ch) => /[A-Z]/.test(ch)).map((ch) => alphabet.indexOf(ch) + 1).join(' ');
}

function a1z26Decode(text: string): string {
  return text.split(/[^0-9]+/).filter(Boolean).map((token) => alphabet[Number(token) - 1] ?? '').join('');
}

function xorTransform(input: string, key: string): string {
  const keyBytes = key.split('').map((char) => char.charCodeAt(0));
  return Array.from(input).map((char, index) => String.fromCharCode(char.charCodeAt(0) ^ keyBytes[index % keyBytes.length])).join('');
}

function vernamTransform(input: string, key: string): string {
  return xorTransform(input, key);
}

function baconianEncode(text: string): string {
  const baconMap: Record<string, string> = {
    A: 'AAAAA', B: 'AAAAB', C: 'AAABA', D: 'AAABB', E: 'AABAA', F: 'AABAB', G: 'AABBA', H: 'AABBB',
    I: 'ABAAA', J: 'ABAAA', K: 'ABAAB', L: 'ABABA', M: 'ABABB', N: 'ABBAA', O: 'ABBAB', P: 'ABBBA',
    Q: 'ABBBB', R: 'BAAAA', S: 'BAAAB', T: 'BAABA', U: 'BAABB', V: 'BABAA', W: 'BABAB', X: 'BABBA',
    Y: 'BABBB', Z: 'BBAAA'
  };
  return Array.from(text.toUpperCase()).map((char) => (/[A-Z]/.test(char) ? baconMap[char] : char)).join(' ');
}

function baconianDecode(text: string): string {
  const baconMap = new Map<string, string>(Object.entries({
    AAAAA: 'A', AAAAB: 'B', AAABA: 'C', AAABB: 'D', AABAA: 'E', AABAB: 'F', AABBA: 'G', AABBB: 'H', ABAAA: 'I', ABAAB: 'K',
    ABABA: 'L', ABABB: 'M', ABBAA: 'N', ABBAB: 'O', ABBBA: 'P', ABBBB: 'Q', BAAAA: 'R', BAAAB: 'S', BAABA: 'T', BAABB: 'U',
    BABAA: 'V', BABAB: 'W', BABBA: 'X', BABBB: 'Y', BBAAA: 'Z'
  }));
  const cleaned = text.replace(/[^AB]/g, ' ');
  const groups = cleaned.split(/\s+/).filter(Boolean);
  return groups.map((group) => baconMap.get(group) ?? '').join('');
}

function tapCodeEncode(text: string): string {
  const tap = [['A', 'B', 'C', 'D', 'E'], ['F', 'G', 'H', 'I', 'K'], ['L', 'M', 'N', 'O', 'P'], ['Q', 'R', 'S', 'T', 'U'], ['V', 'W', 'X', 'Y', 'Z']];
  return Array.from(text.toUpperCase()).map((char) => {
    if (char === 'J') char = 'I';
    for (let row = 0; row < tap.length; row += 1) {
      const col = tap[row].indexOf(char);
      if (col !== -1) return `${row + 1}${col + 1}`;
    }
    return char;
  }).join(' ');
}

function tapCodeDecode(text: string): string {
  const tap = [['A', 'B', 'C', 'D', 'E'], ['F', 'G', 'H', 'I', 'K'], ['L', 'M', 'N', 'O', 'P'], ['Q', 'R', 'S', 'T', 'U'], ['V', 'W', 'X', 'Y', 'Z']];
  const tokens = text.split(/\s+/).filter(Boolean);
  const out: string[] = [];
  for (const token of tokens) {
    if (!/^[1-5][1-5]$/.test(token)) {
      out.push(token);
      continue;
    }
    const row = Number(token[0]) - 1;
    const col = Number(token[1]) - 1;
    out.push(tap[row][col] || '');
  }
  return out.join('');
}

function playfairTransform(input: string, key: string, decrypt = false): string {
  const sanitized = key.toUpperCase().replace(/[^A-Z]/g, '').replace(/J/g, 'I');
  const squareChars = Array.from(new Set(sanitized + alphabet.replace(/J/g, 'I'))).join('');
  const grid = Array.from({ length: 5 }, (_, row) => Array.from({ length: 5 }, (_, col) => squareChars[row * 5 + col]));

  const coords = new Map<string, [number, number]>();
  for (let i = 0; i < 5; i += 1) {
    for (let j = 0; j < 5; j += 1) {
      coords.set(grid[i][j], [i, j]);
    }
  }

  const normalized = input.toUpperCase().replace(/J/g, 'I').replace(/[^A-Z]/g, '').split('').filter(Boolean);
  const pairs: string[] = [];
  for (let i = 0; i < normalized.length; i += 2) {
    if (i + 1 === normalized.length) normalized.push(normalized[i]);
    const first = normalized[i];
    const second = normalized[i + 1];
    if (first === second) {
      pairs.push(first + 'X');
      pairs.push('X' + second);
    } else {
      pairs.push(first + second);
    }
  }

  const output: string[] = [];
  for (const pair of pairs) {
    const [r1, c1] = coords.get(pair[0]) ?? [0, 0];
    const [r2, c2] = coords.get(pair[1]) ?? [0, 0];
    if (r1 === r2) {
      output.push(grid[r1][decrypt ? (c1 - 1 + 5) % 5] + grid[r2][decrypt ? (c2 - 1 + 5) % 5]);
    } else if (c1 === c2) {
      output.push(grid[decrypt ? (r1 - 1 + 5) % 5][c1] + grid[decrypt ? (r2 - 1 + 5) % 5][c2]);
    } else {
      output.push(grid[r1][c2] + grid[r2][c1]);
    }
  }
  return output.join('');
}

function polybiusEncode(text: string): string {
  const grid = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'.replace(/J/g, 'I');
  const flat = grid.split('');
  const rows: string[] = [];
  for (const char of text.toUpperCase()) {
    if (!/[A-Z]/.test(char)) continue;
    const index = flat.indexOf(char === 'J' ? 'I' : char);
    const row = Math.floor(index / 5) + 1;
    const col = (index % 5) + 1;
    rows.push(`${row}${col}`);
  }
  return rows.join(' ');
}

function polybiusDecode(text: string): string {
  const grid = 'ABCDEFGHIJKLMNOPQRSTUVWXYZ'.replace(/J/g, 'I');
  const tokens = text.split(/\s+/).filter(Boolean);
  let out = '';
  for (const token of tokens) {
    if (token.length < 2) continue;
    for (let i = 0; i < token.length; i += 2) {
      const pair = token.slice(i, i + 2);
      const row = Number(pair[0]) - 1;
      const col = Number(pair[1]) - 1;
      const index = row * 5 + col;
      out += grid[index] ?? '';
    }
  }
  return out;
}

function reverseTransform(text: string): string {
  return text.split('').reverse().join('');
}

function sortByScore(a: { score: number }, b: { score: number }) {
  return b.score - a.score;
}

function scoreEnglish(text: string): number {
  const letters = [...text.toUpperCase()].filter((char) => /[A-Z]/.test(char));
  if (!letters.length) return 0;
  let score = 0;
  for (const char of letters) {
    score += englishFrequencies[char] ?? 0.05;
  }
  const commonHits = commonWords.filter((word) => text.toUpperCase().includes(word)).length;
  return score / letters.length + commonHits * 2;
}

function computeIoC(text: string): number {
  const letters = [...text.toUpperCase()].filter((char) => /[A-Z]/.test(char));
  const counts = new Map<string, number>();
  for (const char of letters) {
    counts.set(char, (counts.get(char) ?? 0) + 1);
  }
  const total = letters.length;
  let numerator = 0;
  for (const count of counts.values()) {
    numerator += count * (count - 1);
  }
  return total > 1 ? numerator / (total * (total - 1)) : 0;
}

function computeChiSquared(text: string): number {
  const letters = [...text.toUpperCase()].filter((char) => /[A-Z]/.test(char));
  if (!letters.length) return 0;
  const obs = new Map<string, number>();
  for (const char of letters) {
    obs.set(char, (obs.get(char) ?? 0) + 1);
  }

  const total = letters.length;
  let chi = 0;
  for (const letter of alphabet.split('')) {
    const expected = total * ((englishFrequencies[letter] ?? 0.05) / 100);
    const actual = obs.get(letter) ?? 0;
    chi += ((actual - expected) ** 2) / expected;
  }
  return chi;
}

function buildFrequencyBars(text: string) {
  const letters = [...text.toUpperCase()].filter((char) => /[A-Z]/.test(char));
  const counts = Array.from({ length: 26 }, (_, i) => {
    const letter = alphabet[i];
    const count = letters.filter((value) => value === letter).length;
    const pct = Math.max(4, (count / Math.max(letters.length, 1)) * 100);
    return { letter, count, pct };
  });
  return counts;
}

function getBigrams(text: string): Array<{ pair: string; count: number }> {
  const letters = toAlphabetLetters(text);
  const map = new Map<string, number>();
  for (let i = 0; i < letters.length - 1; i += 1) {
    const pair = letters.slice(i, i + 2);
    map.set(pair, (map.get(pair) ?? 0) + 1);
  }
  return Array.from(map.entries()).map(([pair, count]) => ({ pair, count })).sort((a, b) => b.count - a.count).slice(0, 10);
}

function getTrigrams(text: string): Array<{ trigram: string; count: number }> {
  const letters = toAlphabetLetters(text);
  const map = new Map<string, number>();
  for (let i = 0; i < letters.length - 2; i += 1) {
    const tri = letters.slice(i, i + 3);
    map.set(tri, (map.get(tri) ?? 0) + 1);
  }
  return Array.from(map.entries()).map(([trigram, count]) => ({ trigram, count })).sort((a, b) => b.count - a.count).slice(0, 10);
}

function getWordPatterns(text: string): string[] {
  const words = text.toUpperCase().match(/[A-Z]{3,}/g) ?? [];
  const patterns = new Map<string, number>();
  for (const word of words) {
    const code = Array.from(word).map((char) => {
      const letters = Array.from(word);
      const first = letters.indexOf(char);
      return first === letters.lastIndexOf(char) ? 'A' : 'B';
    }).join(' ');
    patterns.set(code, (patterns.get(code) ?? 0) + 1);
  }
  return Array.from(patterns.entries()).map(([pattern, count]) => `${pattern} (${count})`).slice(0, 8);
}

function getCandidates(ciphertext: string) {
  const sanitized = toAlphabetLetters(ciphertext);
  const candidates: Array<{ label: string; score: number; text: string }> = [];

  for (let shift = 0; shift < 26; shift += 1) {
    const decoded = caesarTransform(sanitized, shift, true);
    candidates.push({ label: `Caesar ${shift}`, score: scoreEnglish(decoded), text: decoded });
  }

  candidates.push({ label: 'Atbash', score: scoreEnglish(atbashTransform(sanitized)), text: atbashTransform(sanitized) });
  for (const a of [1, 3, 5, 7, 9, 11, 15, 17, 19, 21, 23, 25]) {
    const output = affineTransform(sanitized, a, 3, true);
    candidates.push({ label: `Affine (${a}, 3)`, score: scoreEnglish(output), text: output });
  }
  for (const rails of [2, 3, 4, 5, 6]) {
    const output = railFenceTransform(sanitized, rails, true);
    candidates.push({ label: `Rail Fence ${rails}`, score: scoreEnglish(output), text: output });
  }
  for (const key of ['SECRET', 'GCHQ', 'CIPHER', 'CRYPTO', 'KEY', 'MATRIX', 'ALPHA']) {
    const output = vigenereTransform(sanitized, key, true);
    candidates.push({ label: `Vigenère ${key}`, score: scoreEnglish(output), text: output });
  }

  return candidates.sort(sortByScore).slice(0, 8);
}

function clamp(value: number, min: number, max: number) {
  return Math.min(Math.max(value, min), max);
}

function createImageCanvas(width: number, height: number): HTMLCanvasElement {
  const canvas = document.createElement('canvas');
  canvas.width = width;
  canvas.height = height;
  return canvas;
}

const quickCipherMap: Record<string, (text: string, mode: 'encrypt' | 'decrypt', key?: string) => string> = {
  Caesar: (text, mode, key = '3') => caesarTransform(text, Number(key) || 3, mode === 'decrypt'),
  ROT13: (text) => rot13(text),
  ROT47: (text) => rot47(text),
  Atbash: (text) => atbashTransform(text),
  Affine: (text, mode) => affineTransform(text, 5, 8, mode === 'decrypt'),
  Polybius: (text, mode) => (mode === 'encrypt' ? polybiusEncode(text) : polybiusDecode(text)),
  Morse: (text, mode) => (mode === 'encrypt' ? morseEncode(text) : morseDecode(text)),
  A1Z26: (text, mode) => (mode === 'encrypt' ? a1z26Encode(text) : a1z26Decode(text)),
  Binary: (text, mode) => (mode === 'encrypt' ? binaryEncode(text) : binaryDecode(text)),
  Hex: (text, mode) => (mode === 'encrypt' ? hexEncode(text) : hexDecode(text)),
  Octal: (text, mode) => (mode === 'encrypt' ? octalEncode(text) : octalDecode(text)),
  Base64: (text, mode) => (mode === 'encrypt' ? base64Encode(text) : base64Decode(text)),
  Base32: (text, mode) => (mode === 'encrypt' ? base32Encode(text) : text),
  XOR: (text, mode, key = 'GCHQ') => xorTransform(text, key),
  Vernam: (text, mode, key = 'GCHQ') => vernamTransform(text, key),
  Baconian: (text, mode) => (mode === 'encrypt' ? baconianEncode(text) : baconianDecode(text)),
  TapCode: (text, mode) => (mode === 'encrypt' ? tapCodeEncode(text) : tapCodeDecode(text)),
  Vigenère: (text, mode, key = 'SECRET') => vigenereTransform(text, key, mode === 'decrypt'),
  Beaufort: (text, mode, key = 'SECRET') => beaufordTransform(text, key, mode === 'decrypt'),
  Reverser: (text) => reverseTransform(text),
  RailFence: (text, mode) => railFenceTransform(text, 3, mode === 'decrypt'),
  Columnar: (text) => text,
  Anagram: (text) => text,
  'Myszkowski': (text) => text,
  'Fleissner Grille': (text) => text,
  Playfair: (text, mode, key = 'CIPHER') => playfairTransform(text, key, mode === 'decrypt'),
  'Two-Square': (text) => text,
  'Four-Square': (text) => text,
  Enigma: (text) => text,
  'M-209': (text) => text,
  'Jefferson Disk': (text) => text,
  'Lorenz SZ40/42': (text) => text,
  'Alberti Disk': (text) => text,
  'Book Cipher': (text) => text
};

function App() {
  const [activeTab, setActiveTab] = useState<'workspace' | 'solver' | 'crypto' | 'stego'>('workspace');
  const [themeKey, setThemeKey] = useState<ThemeKey>(() => {
    const saved = localStorage.getItem('ciphernexus-theme');
    return (saved as ThemeKey) || defaultTheme;
  });
  const [primaryColor, setPrimaryColor] = useState('#4cc9f0');
  const [fontMode, setFontMode] = useState<FontMode>('sans');
  const [layoutMode, setLayoutMode] = useState<LayoutMode>('split');
  const [selectedAlgorithm, setSelectedAlgorithm] = useState('Caesar');
  const [selectedGroup, setSelectedGroup] = useState<CipherGroup>('Substitution');
  const [inputText, setInputText] = useState('THE QUICK BROWN FOX JUMPS OVER THE LAZY DOG');
  const [outputText, setOutputText] = useState('');
  const [keyValue, setKeyValue] = useState('3');
  const [solverText, setSolverText] = useState('QEBI JQXQJXKX');
  const [substitutionMap, setSubstitutionMap] = useState('A->D, E->T, O->E, H->A');

  const fileInputRef = useRef<HTMLInputElement | null>(null);
  const canvasRef = useRef<HTMLCanvasElement | null>(null);
  const imageRef = useRef<HTMLImageElement | null>(null);
  const [loadedImageUrl, setLoadedImageUrl] = useState<string | null>(null);
  const [zoom, setZoom] = useState(100);
  const [channelFilter, setChannelFilter] = useState<'all' | 'r' | 'g' | 'b' | 'a'>('all');
  const [invert, setInvert] = useState(false);
  const [contrastBoost, setContrastBoost] = useState(0);
  const [lsbResult, setLsbResult] = useState('No payload extracted yet.');
  const [bitPlanes, setBitPlanes] = useState<Array<{ id: string; canvas: HTMLCanvasElement }>>([]);

  const theme = themeDefinitions[themeKey];

  useEffect(() => {
    const root = document.documentElement;
    root.style.setProperty('--primary', theme.primary);
    root.style.setProperty('--accent', theme.accent);
    root.style.setProperty('--bg', theme.bg);
    root.style.setProperty('--panel', theme.panel);
    root.style.setProperty('--text', theme.text);
    root.style.setProperty('--success', theme.success);
    root.style.setProperty('--font-sans', fontMode === 'mono' ? 'SFMono-Regular' : fontMode === 'serif' ? 'Georgia' : 'Inter');
    document.body.style.background = theme.bg;
    document.body.style.color = theme.text;
    localStorage.setItem('ciphernexus-theme', themeKey);
    localStorage.setItem('ciphernexus-primary', primaryColor);
    localStorage.setItem('ciphernexus-font', fontMode);
    localStorage.setItem('ciphernexus-layout', layoutMode);
  }, [themeKey, primaryColor, fontMode, layoutMode, theme]);

  useEffect(() => {
    const savedPrimary = localStorage.getItem('ciphernexus-primary');
    const savedFont = localStorage.getItem('ciphernexus-font');
    const savedLayout = localStorage.getItem('ciphernexus-layout');
    if (savedPrimary) setPrimaryColor(savedPrimary);
    if (savedFont) setFontMode(savedFont as FontMode);
    if (savedLayout) setLayoutMode(savedLayout as LayoutMode);
  }, []);

  const generatedPalette = useMemo(() => ({
    '--primary': primaryColor,
    '--accent': adjustColor(primaryColor, 24),
    '--text': theme.text,
    '--bg': theme.bg,
    '--panel': theme.panel,
    '--panel-strong': theme.panel
  }), [primaryColor, theme]);

  const filteredAlgorithms = algorithmList.filter((item) => item.group === selectedGroup || selectedGroup === 'Substitution' ? item.group === 'Substitution' : true);

  const updateOutput = (mode: 'encrypt' | 'decrypt') => {
    const selected = algorithmList.find((item) => item.label === selectedAlgorithm) ?? algorithmList[0];
    const fn = quickCipherMap[selected.label] ?? ((text: string) => text);
    const transformed = fn(inputText, mode, keyValue);
    setOutputText(transformed);
  };

  const copyOutput = () => navigator.clipboard.writeText(outputText || inputText);

  const applyManualSubstitution = () => {
    const map: Record<string, string> = {};
    for (const pair of substitutionMap.split(',')) {
      const [from, to] = pair.trim().split('->');
      if (from && to) map[from.trim().toUpperCase()] = to.trim().toUpperCase();
    }
    if (!Object.keys(map).length) {
      setOutputText(solverText);
      return;
    }

    const result = solverText.toUpperCase().split('').map((char) => {
      if (!/[A-Z]/.test(char)) return char;
      return map[char] ?? char;
    }).join('');
    setOutputText(result);
  };

  const topCandidates = useMemo(() => getCandidates(solverText), [solverText]);
  const frequencyBars = useMemo(() => buildFrequencyBars(solverText), [solverText]);
  const bigrams = useMemo(() => getBigrams(solverText), [solverText]);
  const trigrams = useMemo(() => getTrigrams(solverText), [solverText]);
  const wordPatterns = useMemo(() => getWordPatterns(solverText), [solverText]);
  const iocValue = useMemo(() => computeIoC(solverText), [solverText]);
  const chiValue = useMemo(() => computeChiSquared(solverText), [solverText]);

  const handleImageUpload = (event: React.ChangeEvent<HTMLInputElement>) => {
    const file = event.target.files?.[0];
    if (!file) return;
    const url = URL.createObjectURL(file);
    setLoadedImageUrl(url);
    const img = new Image();
    img.onload = () => {
      imageRef.current = img;
      const canvas = canvasRef.current;
      if (!canvas) return;
      const ctx = canvas.getContext('2d');
      if (!ctx) return;
      const width = img.width;
      const height = img.height;
      canvas.width = width;
      canvas.height = height;
      ctx.clearRect(0, 0, width, height);
      ctx.drawImage(img, 0, 0, width, height);
      renderBitPlanes();
      extractLsbText();
    };
    img.src = url;
  };

  const drawProcessedCanvas = () => {
    const canvas = canvasRef.current;
    const img = imageRef.current;
    if (!canvas || !img) return;
    const ctx = canvas.getContext('2d');
    if (!ctx) return;
    const width = img.width;
    const height = img.height;
    canvas.width = width;
    canvas.height = height;
    ctx.clearRect(0, 0, width, height);
    ctx.drawImage(img, 0, 0, width, height);

    const imageData = ctx.getImageData(0, 0, width, height);
    const data = imageData.data;
    for (let i = 0; i < data.length; i += 4) {
      let r = data[i];
      let g = data[i + 1];
      let b = data[i + 2];
      let a = data[i + 3];

      if (channelFilter === 'r') {
        g = r; b = r; a = 255;
      } else if (channelFilter === 'g') {
        r = g; b = g; a = 255;
      } else if (channelFilter === 'b') {
        r = b; g = b; a = 255;
      } else if (channelFilter === 'a') {
        r = a; g = a; b = a;
      }

      if (invert) {
        r = 255 - r;
        g = 255 - g;
        b = 255 - b;
      }

      if (contrastBoost > 0) {
        const factor = (259 * (contrastBoost + 255)) / (255 * (259 - contrastBoost));
        r = clamp(factor * (r - 128) + 128, 0, 255);
        g = clamp(factor * (g - 128) + 128, 0, 255);
        b = clamp(factor * (b - 128) + 128, 0, 255);
      }

      data[i] = r;
      data[i + 1] = g;
      data[i + 2] = b;
      data[i + 3] = a;
    }
    ctx.putImageData(imageData, 0, 0);
    renderBitPlanes();
    extractLsbText();
  };

  const renderBitPlanes = () => {
    const canvas = canvasRef.current;
    if (!canvas) return;
    const ctx = canvas.getContext('2d');
    if (!ctx) return;
    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;
    const planes: Array<{ id: string; canvas: HTMLCanvasElement }> = [];

    for (let bit = 0; bit < 8; bit += 1) {
      const newCanvas = createImageCanvas(canvas.width, canvas.height);
      const bitCtx = newCanvas.getContext('2d');
      if (!bitCtx) continue;
      const next = bitCtx.createImageData(canvas.width, canvas.height);
      for (let i = 0; i < data.length; i += 4) {
        const red = (data[i] >> bit) & 1 ? 255 : 0;
        const green = (data[i + 1] >> bit) & 1 ? 255 : 0;
        const blue = (data[i + 2] >> bit) & 1 ? 255 : 0;
        next.data[i] = red;
        next.data[i + 1] = green;
        next.data[i + 2] = blue;
        next.data[i + 3] = 255;
      }
      bitCtx.putImageData(next, 0, 0);
      planes.push({ id: `bit-${bit}`, canvas: newCanvas });
    }
    setBitPlanes(planes);
  };

  const extractLsbText = () => {
    const canvas = canvasRef.current;
    if (!canvas) {
      setLsbResult('No payload extracted yet.');
      return;
    }

    const ctx = canvas.getContext('2d');
    if (!ctx) {
      setLsbResult('Canvas unavailable.');
      return;
    }

    const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
    const data = imageData.data;
    let bits = '';
    for (let i = 0; i < data.length; i += 4) {
      bits += (data[i] & 1).toString();
      bits += (data[i + 1] & 1).toString();
      bits += (data[i + 2] & 1).toString();
    }

    const bytes: string[] = [];
    for (let i = 0; i + 8 <= bits.length; i += 8) {
      const chunk = bits.slice(i, i + 8);
      const byte = parseInt(chunk, 2);
      if (byte === 0) continue;
      bytes.push(String.fromCharCode(byte));
    }
    const payload = bytes.join('').replace(/\u0000/g, '');
    setLsbResult(payload.length ? payload.slice(0, 200) : 'No ASCII payload was detected in the LSB stream.');
  };

  useEffect(() => {
    if (imageRef.current && canvasRef.current) {
      drawProcessedCanvas();
    }
  }, [zoom, channelFilter, invert, contrastBoost]);

  const fontClass =
    fontMode === 'mono'
      ? 'font-mono-sans'
      : fontMode === 'serif'
        ? 'font-serif-edu'
        : 'font-sans-ui';

  const tabButtonClass = (tab: typeof activeTab) =>
    `px-4 py-2 rounded-xl text-sm font-medium border ${activeTab === tab ? 'border-transparent text-white' : 'border-white/10 text-slate-300 hover:text-white'}`;

  const rootStyle = {
    background: theme.bg,
    color: theme.text,
    fontFamily: fontMode === 'mono' ? 'SFMono-Regular, ui-monospace, monospace' : fontMode === 'serif' ? 'Georgia, serif' : 'Inter, sans-serif',
    ...generatedPalette
  } as React.CSSProperties;

  return (
    <div className="app-shell" style={rootStyle}>
      <header className="topbar sticky top-0 z-20">
        <div className="mx-auto max-w-7xl px-5 py-4">
          <div className="flex flex-col gap-4 md:flex-row md:items-center md:justify-between">
            <div>
              <div className="text-xs uppercase tracking-[0.35em] text-slate-400">Intelligence Suite</div>
              <h1 className="mt-1 text-3xl font-semibold">CipherNexus GCHQ</h1>
            </div>
            <div className="flex flex-wrap items-center gap-3">
              {(['workspace', 'solver', 'crypto', 'stego'] as const).map((tab) => (
                <button
                  key={tab}
                  className={tabButtonClass(tab)}
                  style={{ background: activeTab === tab ? `linear-gradient(135deg, ${theme.primary}, ${theme.accent})` : 'transparent' }}
                  onClick={() => setActiveTab(tab)}
                >
                  {tab === 'workspace' ? 'Workspace' : tab === 'solver' ? 'Auto Multi-Solver' : tab === 'crypto' ? 'Cryptanalysis' : 'Image Analysis'}
                </button>
              ))}
            </div>
          </div>
        </div>
      </header>

      <main className="mx-auto w-full max-w-7xl px-5 py-6">
        {activeTab === 'workspace' && (
          <div className="space-y-6">
            <section className="card p-5">
              <div className="flex flex-col gap-4 lg:flex-row lg:items-center lg:justify-between">
                <div>
                  <p className="text-xs uppercase tracking-[0.3em] text-slate-400">Cipher Suite</p>
                  <h2 className="mt-2 text-2xl font-semibold">Workspace</h2>
                </div>
                <div className="flex flex-wrap gap-3">
                  <select value={themeKey} onChange={(e) => setThemeKey(e.target.value as ThemeKey)} className="rounded-xl border border-white/10 bg-slate-900/80 px-3 py-2 text-sm text-white">
                    {Object.entries(themeDefinitions).map(([value, themeData]) => (
                      <option key={value} value={value}>{themeData.name}</option>
                    ))}
                  </select>
                  <select value={fontMode} onChange={(e) => setFontMode(e.target.value as FontMode)} className="rounded-xl border border-white/10 bg-slate-900/80 px-3 py-2 text-sm text-white">
                    <option value="sans">Sans</option>
                    <option value="mono">Monospace</option>
                    <option value="serif">Serif</option>
                  </select>
                  <select value={layoutMode} onChange={(e) => setLayoutMode(e.target.value as LayoutMode)} className="rounded-xl border border-white/10 bg-slate-900/80 px-3 py-2 text-sm text-white">
                    <option value="split">Split</option>
                    <option value="stacked">Stacked</option>
                  </select>
                  <input type="color" value={primaryColor} onChange={(e) => setPrimaryColor(e.target.value)} className="h-10 w-14 rounded-xl border border-white/10 bg-transparent p-1" />
                </div>
              </div>
            </section>

            <section className="card p-5">
              <div className="mb-5 flex flex-wrap gap-2">
                {(['Substitution', 'Polyalphabetic', 'Transposition', 'Historical', 'Modern'] as CipherGroup[]).map((group) => (
                  <button
                    key={group}
                    className={`rounded-full border px-4 py-2 text-sm ${selectedGroup === group ? 'border-transparent text-white' : 'border-white/10 text-slate-300'}`}
                    style={{ background: selectedGroup === group ? `linear-gradient(135deg, ${theme.primary}, ${theme.accent})` : 'transparent' }}
                    onClick={() => setSelectedGroup(group)}
                  >
                    {group}
                  </button>
                ))}
              </div>

              <div className="grid gap-4 lg:grid-cols-[260px_1fr]">
                <div className="panel p-3">
                  <div className="mb-3 text-xs uppercase tracking-[0.2em] text-slate-400">Algorithms</div>
                  <div className="max-h-[420px] space-y-2 overflow-auto pr-1">
                    {algorithmList.filter((item) => selectedGroup === 'Substitution' ? item.group === 'Substitution' : item.group === selectedGroup).map((item) => (
                      <button
                        key={item.label}
                        onClick={() => setSelectedAlgorithm(item.label)}
                        className={`w-full rounded-xl border px-3 py-2 text-left text-sm ${selectedAlgorithm === item.label ? 'border-transparent text-white' : 'border-white/10 text-slate-300 hover:border-white/20'}`}
                        style={{ background: selectedAlgorithm === item.label ? `${theme.primary}22` : 'transparent' }}
                      >
                        {item.label}
                      </button>
                    ))}
                  </div>
                </div>

                <div className="space-y-4">
                  <div className="flex flex-wrap items-center gap-2">
                    <div className="word-badge text-xs uppercase tracking-[0.2em]">{selectedAlgorithm}</div>
                    <input
                      value={keyValue}
                      onChange={(e) => setKeyValue(e.target.value)}
                      className="rounded-xl border border-white/10 bg-slate-900/80 px-3 py-2 text-sm text-white"
                      placeholder="Key"
                    />
                  </div>

                  <div className={`grid ${layoutMode === 'split' ? 'grid-cols-2' : 'grid-cols-1'} gap-4`}>
                    <div className="panel p-3">
                      <div className="mb-2 flex items-center justify-between text-xs uppercase tracking-[0.2em] text-slate-400">
                        <span>Input</span>
                        <button className="text-slate-300" onClick={() => setInputText('')}>Clear</button>
                      </div>
                      <textarea value={inputText} onChange={(e) => setInputText(e.target.value)} className={`h-72 w-full rounded-xl border border-white/10 bg-slate-950/70 p-3 text-base ${fontClass}`} />
                    </div>

                    <div className="panel p-3">
                      <div className="mb-2 flex items-center justify-between text-xs uppercase tracking-[0.2em] text-slate-400">
                        <span>Output</span>
                        <button className="text-slate-300" onClick={copyOutput}>Copy</button>
                      </div>
                      <textarea value={outputText} onChange={(e) => setOutputText(e.target.value)} className={`h-72 w-full rounded-xl border border-white/10 bg-slate-950/70 p-3 text-base ${fontClass}`} />
                    </div>
                  </div>

                  <div className="flex flex-wrap gap-3">
                    <button className="rounded-xl px-4 py-2 font-medium text-slate-950" style={{ background: `linear-gradient(135deg, ${theme.primary}, ${theme.accent})` }} onClick={() => updateOutput('encrypt')}>Encrypt</button>
                    <button className="rounded-xl border border-white/10 bg-slate-900/80 px-4 py-2 text-sm text-white" onClick={() => updateOutput('decrypt')}>Decrypt</button>
                    <button className="rounded-xl border border-white/10 bg-slate-900/80 px-4 py-2 text-sm text-white" onClick={() => {
                      const temp = inputText; setInputText(outputText || ''); setOutputText(temp);
                    }}>Swap</button>
                  </div>
                </div>
              </div>
            </section>
          </div>
        )}

        {activeTab === 'solver' && (
          <div className="space-y-6">
            <section className="card p-5">
              <div className="flex flex-col gap-4 lg:flex-row lg:items-center lg:justify-between">
                <div>
                  <p className="text-xs uppercase tracking-[0.3em] text-slate-400">Automated Assist</p>
                  <h2 className="mt-2 text-2xl font-semibold">Auto Multi-Solver</h2>
                </div>
                <div className="word-badge px-3 py-2 text-xs uppercase tracking-[0.2em]">Ranking by English plausibility</div>
              </div>
            </section>

            <section className="grid gap-6 lg:grid-cols-[1.1fr_0.9fr]">
              <div className="card p-5">
                <div className="mb-3 text-xs uppercase tracking-[0.3em] text-slate-400">Ciphertext</div>
                <textarea value={solverText} onChange={(e) => setSolverText(e.target.value)} className={`h-64 w-full rounded-xl border border-white/10 bg-slate-950/70 p-4 text-lg ${fontClass}`} />
                <div className="mt-3 flex flex-wrap gap-3">
                  <button className="rounded-xl px-4 py-2 font-medium text-slate-950" style={{ background: `linear-gradient(135deg, ${theme.primary}, ${theme.accent})` }} onClick={() => setSolverText('QEBI JQXQJXKX')}>Load Sample</button>
                  <button className="rounded-xl border border-white/10 bg-slate-900/80 px-4 py-2 text-sm text-white" onClick={() => setSolverText('')}>Clear</button>
                </div>
              </div>

              <div className="card p-5">
                <div className="mb-3 text-xs uppercase tracking-[0.3em] text-slate-400">Candidate Ranking</div>
                <div className="space-y-3">
                  {topCandidates.map((candidate, index) => (
                    <div key={`${candidate.label}-${index}`} className="rounded-2xl border border-white/10 bg-slate-950/40 p-3">
                      <div className="flex items-center justify-between gap-3">
                        <div className="font-medium">#{index + 1} • {candidate.label}</div>
                        <div className="text-xs text-slate-400">score {candidate.score.toFixed(1)}</div>
                      </div>
                      <div className="mt-2 whitespace-pre-wrap text-sm text-slate-200">{candidate.text}</div>
                    </div>
                  ))}
                </div>
              </div>
            </section>
          </div>
        )}

        {activeTab === 'crypto' && (
          <div className="space-y-6">
            <section className="card p-5">
              <div className="flex flex-col gap-4 lg:flex-row lg:items-center lg:justify-between">
                <div>
                  <p className="text-xs uppercase tracking-[0.3em] text-slate-400">Signal Analysis</p>
                  <h2 className="mt-2 text-2xl font-semibold">Cryptanalysis Dashboard</h2>
                </div>
                <div className="word-badge px-3 py-2 text-xs uppercase tracking-[0.2em]">English Frequency + Pattern Matching</div>
              </div>
            </section>

            <section className="grid gap-6 lg:grid-cols-[1.1fr_0.9fr]">
              <div className="space-y-6">
                <div className="card p-5">
                  <div className="mb-4 text-xs uppercase tracking-[0.3em] text-slate-400">Frequency Analysis</div>
                  <div className="bar-chart">
                    {frequencyBars.map((bar) => (
                      <div key={bar.letter} className="flex h-full flex-1 flex-col items-center justify-end gap-2">
                        <div className="w-full rounded-t-lg border border-white/10 bg-gradient-to-t from-sky-600 to-cyan-400" style={{ height: `${bar.pct}%` }} />
                        <span className="text-[10px] text-slate-400">{bar.letter}</span>
                      </div>
                    ))}
                  </div>
                </div>

                <div className="card p-5">
                  <div className="mb-4 text-xs uppercase tracking-[0.3em] text-slate-400">Pattern Profiles</div>
                  <div className="grid gap-4 md:grid-cols-2">
                    <div>
                      <div className="mb-2 text-sm font-medium text-slate-300">Bigrams</div>
                      <div className="space-y-2">
                        {bigrams.map((item) => (
                          <div key={item.pair} className="flex items-center justify-between rounded-xl border border-white/10 bg-slate-950/40 px-3 py-2 text-sm">
                            <span>{item.pair}</span>
                            <span className="text-slate-400">{item.count}</span>
                          </div>
                        ))}
                      </div>
                    </div>
                    <div>
                      <div className="mb-2 text-sm font-medium text-slate-300">Trigrams</div>
                      <div className="space-y-2">
                        {trigrams.map((item) => (
                          <div key={item.trigram} className="flex items-center justify-between rounded-xl border border-white/10 bg-slate-950/40 px-3 py-2 text-sm">
                            <span>{item.trigram}</span>
                            <span className="text-slate-400">{item.count}</span>
                          </div>
                        ))}
                      </div>
                    </div>
                  </div>
                </div>
              </div>

              <div className="space-y-6">
                <div className="card p-5">
                  <div className="mb-3 text-xs uppercase tracking-[0.3em] text-slate-400">Metrics</div>
                  <div className="space-y-4">
                    <div>
                      <div className="mb-1 flex items-center justify-between text-sm">
                        <span>Index of Coincidence</span>
                        <span className="text-slate-300">{iocValue.toFixed(4)}</span>
                      </div>
                      <div className="h-2.5 overflow-hidden rounded-full bg-slate-800">
                        <div className="meter-fill h-full rounded-full" style={{ width: `${Math.min(100, (iocValue / 0.1) * 100)}%` }} />
                      </div>
                    </div>

                    <div>
                      <div className="mb-1 flex items-center justify-between text-sm">
                        <span>Chi-Squared</span>
                        <span className="text-slate-300">{chiValue.toFixed(2)}</span>
                      </div>
                      <div className="h-2.5 overflow-hidden rounded-full bg-slate-800">
                        <div className="meter-fill h-full rounded-full" style={{ width: `${Math.min(100, (chiValue / 25) * 100)}%` }} />
                      </div>
                    </div>
                  </div>
                </div>

                <div className="card p-5">
                  <div className="mb-3 text-xs uppercase tracking-[0.3em] text-slate-400">Manual Substitution</div>
                  <textarea value={solverText} onChange={(e) => setSolverText(e.target.value)} className={`h-34 w-full rounded-xl border border-white/10 bg-slate-950/70 p-4 text-base ${fontClass}`} />
                  <input value={substitutionMap} onChange={(e) => setSubstitutionMap(e.target.value)} className="mt-3 w-full rounded-xl border border-white/10 bg-slate-950/70 px-3 py-2 text-sm text-white" placeholder="A->D, E->T, O->E" />
                  <button className="mt-3 rounded-xl px-4 py-2 font-medium text-slate-950" style={{ background: `linear-gradient(135deg, ${theme.primary}, ${theme.accent})` }} onClick={applyManualSubstitution}>Apply Mapping</button>
                  <div className="mt-4 space-y-2">
                    {wordPatterns.map((pattern) => (
                      <div key={pattern} className="rounded-xl border border-white/10 bg-slate-950/50 px-3 py-2 text-sm text-slate-300">{pattern}</div>
                    ))}
                  </div>
                  <div className="mt-4 rounded-2xl border border-white/10 bg-slate-950/60 p-3 text-sm text-slate-300">
                    Output: {outputText || 'No substitution applied yet.'}
                  </div>
                </div>
              </div>
            </section>
          </div>
        )}

        {activeTab === 'stego' && (
          <div className="space-y-6">
            <section className="card p-5">
              <div className="flex flex-col gap-4 lg:flex-row lg:items-center lg:justify-between">
                <div>
                  <p className="text-xs uppercase tracking-[0.3em] text-slate-400">Image & Payload Forensics</p>
                  <h2 className="mt-2 text-2xl font-semibold">Steganography Toolkit</h2>
                </div>
                <button className="rounded-xl px-4 py-2 font-medium text-slate-950" style={{ background: `linear-gradient(135deg, ${theme.primary}, ${theme.accent})` }} onClick={() => fileInputRef.current?.click()}>Upload Image</button>
                <input ref={fileInputRef} type="file" accept="image/*" className="hidden" onChange={handleImageUpload} />
              </div>
            </section>

            <section className="grid gap-6 lg:grid-cols-[1.2fr_0.8fr]">
              <div className="card p-5">
                <div className="mb-4 flex items-center justify-between text-xs uppercase tracking-[0.2em] text-slate-400">
                  <span>Canvas Viewer</span>
                  <span>{zoom}% zoom</span>
                </div>

                <div className="canvas-wrap">
                  <canvas ref={canvasRef} style={{ transform: `scale(${zoom / 100})`, transformOrigin: 'center center' }} />
                </div>

                <div className="mt-4 flex flex-col gap-3 md:flex-row md:items-center md:justify-between">
                  <input type="range" min={50} max={200} value={zoom} onChange={(e) => setZoom(Number(e.target.value))} className="w-full md:max-w-xs" />
                  <div className="flex flex-wrap gap-2">
                    {(['all', 'r', 'g', 'b', 'a'] as const).map((mode) => (
                      <button key={mode} className={`rounded-full border px-3 py-1.5 text-xs ${channelFilter === mode ? 'border-transparent text-white' : 'border-white/10 text-slate-300'}`} style={{ background: channelFilter === mode ? `linear-gradient(135deg, ${theme.primary}, ${theme.accent})` : 'transparent' }} onClick={() => setChannelFilter(mode)}>{mode.toUpperCase()}</button>
                    ))}
                  </div>
                </div>

                <div className="mt-4 flex flex-wrap gap-2">
                  <button className="rounded-xl border border-white/10 bg-slate-900/80 px-3 py-2 text-sm text-white" onClick={() => setInvert((previous) => !previous)}>Invert</button>
                  <button className="rounded-xl border border-white/10 bg-slate-900/80 px-3 py-2 text-sm text-white" onClick={() => setContrastBoost((prev) => (prev < 100 ? prev + 25 : 0))}>Contrast +{contrastBoost}</button>
                  <button className="rounded-xl border border-white/10 bg-slate-900/80 px-3 py-2 text-sm text-white" onClick={() => setContrastBoost(0)}>Reset Filters</button>
                </div>
              </div>

              <div className="space-y-6">
                <div className="card p-5">
                  <div className="mb-3 text-xs uppercase tracking-[0.3em] text-slate-400">LSB Extractor</div>
                  <div className="rounded-2xl border border-white/10 bg-slate-950/50 p-4 text-sm text-slate-200">{lsbResult}</div>
                  <button className="mt-3 rounded-xl px-4 py-2 font-medium text-slate-950" style={{ background: `linear-gradient(135deg, ${theme.primary}, ${theme.accent})` }} onClick={extractLsbText}>Extract Payload</button>
                </div>

                <div className="card p-5">
                  <div className="mb-3 text-xs uppercase tracking-[0.3em] text-slate-400">Bit-Plane Slicing</div>
                  <div className="bitplane-grid">
                    {bitPlanes.slice(0, 8).map((plane) => (
                      <div key={plane.id} className="bitplane-card">
                        <div className="mb-2 text-center text-[11px] uppercase tracking-[0.2em] text-slate-400">{plane.id}</div>
                        <canvas width={120} height={120} onClick={() => {}} ref={(node) => {
                          if (node && plane.canvas) {
                            const ctx = node.getContext('2d');
                            if (ctx) {
                              ctx.clearRect(0, 0, node.width, node.height);
                              ctx.drawImage(plane.canvas, 0, 0, node.width, node.height);
                            }
                          }
                        }} />
                      </div>
                    ))}
                  </div>
                </div>
              </div>
            </section>
          </div>
        )}
      </main>
    </div>
  );
}

function adjustColor(hex: string, amount: number) {
  const clean = hex.replace('#', '');
  const parsed = Number.parseInt(clean, 16);
  const r = clamp(((parsed >> 16) & 255) + amount, 0, 255);
  const g = clamp(((parsed >> 8) & 255) + amount, 0, 255);
  const b = clamp((parsed & 255) + amount, 0, 255);
  return `#${[r, g, b].map((value) => value.toString(16).padStart(2, '0')).join('')}`;
}

export default App;
