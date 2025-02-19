Merged [pull request #105](https://github.com/antelle/node-stream-zip/pull/105) to support UTF-8 entry name.
Merged [pull request #104](https://github.com/antelle/node-stream-zip/pull/104) and [pull request #93](https://github.com/antelle/node-stream-zip/pull/93) to fix some bugs.

---

# node-stream-zip ![CI Checks](https://github.com/antelle/node-stream-zip/workflows/CI%20Checks/badge.svg)

node.js library for reading and extraction of ZIP archives.  
Features:

- it never loads entire archive into memory, everything is read by chunks
- large archives support
- all operations are non-blocking, no sync i/o
- fast initialization
- no dependencies, no binary addons
- decompression with built-in zlib module
- deflate, sfx, macosx/windows built-in archives
- ZIP64 support

## Installation

```sh
npm i node-stream-zip
```

## Usage

There are two APIs provided:
1. [promise-based / async](#async-api) 
2. [callbacks](#callback-api)

It's recommended to use the new, promise API, however the legacy callback API 
may be more flexible for certain operations.

### Async API

Open a zip file
```javascript
const StreamZip = require('node-stream-zip');
const zip = new StreamZip.async({ file: 'archive.zip' });
```

Stream one entry to stdout
```javascript
const stm = await zip.stream('path/inside/zip.txt');
stm.pipe(process.stdout);
stm.on('end', () => zip.close());
```

Read a file as buffer
```javascript
const data = await zip.entryData('path/inside/zip.txt');
await zip.close();
```

Extract one file to disk
```javascript
await zip.extract('path/inside/zip.txt', './extracted.txt');
await zip.close();
```

List entries
```javascript
const entriesCount = await zip.entriesCount;
console.log(`Entries read: ${entriesCount}`);

const entries = await zip.entries();
for (const entry of Object.values(entries)) {
    const desc = entry.isDirectory ? 'directory' : `${entry.size} bytes`;
    console.log(`Entry ${entry.name}: ${desc}`);
}

// Do not forget to close the file once you're done
await zip.close();
```

Extract a folder from archive to disk
```javascript
fs.mkdirSync('extracted');
await zip.extract('path/inside/zip/', './extracted');
await zip.close();
```

Extract everything
```javascript
fs.mkdirSync('extracted');
const count = await zip.extract(null, './extracted');
console.log(`Extracted ${count} entries`);
await zip.close();
```

When extracting a folder, you can listen to `extract` event
```javascript
zip.on('extract', (entry, file) => {
    console.log(`Extracted ${entry.name} to ${file}`);
});
```

`entry` event is generated for every entry during loading
```javascript
zip.on('entry', entry => {
    // you can already stream this entry,
    // without waiting until all entry descriptions are read (suitable for very large archives)
    console.log(`Read entry ${entry.name}`);
});
```

### Callback API

Open a zip file
```javascript
const StreamZip = require('node-stream-zip');
const zip = new StreamZip({ file: 'archive.zip' });

// Handle errors
zip.on('error', err => { /*...*/ });
```

List entries
```javascript
zip.on('ready', () => {
    console.log('Entries read: ' + zip.entriesCount);
    for (const entry of Object.values(zip.entries())) {
        const desc = entry.isDirectory ? 'directory' : `${entry.size} bytes`;
        console.log(`Entry ${entry.name}: ${desc}`);
    }
    // Do not forget to close the file once you're done
    zip.close();
});
```

Stream one entry to stdout
```javascript
zip.on('ready', () => {
    zip.stream('path/inside/zip.txt', (err, stm) => {
        stm.pipe(process.stdout);
        stm.on('end', () => zip.close());
    });
});
```

Extract one file to disk
```javascript
zip.on('ready', () => {
    zip.extract('path/inside/zip.txt', './extracted.txt', err => {
        console.log(err ? 'Extract error' : 'Extracted');
        zip.close();
    });
});
```

Extract a folder from archive to disk
```javascript
zip.on('ready', () => {
    fs.mkdirSync('extracted');
    zip.extract('path/inside/zip/', './extracted', err => {
        console.log(err ? 'Extract error' : 'Extracted');
        zip.close();
    });
});
```

Extract everything
```javascript
zip.on('ready', () => {
    fs.mkdirSync('extracted');
    zip.extract(null, './extracted', (err, count) => {
        console.log(err ? 'Extract error' : `Extracted ${count} entries`);
        zip.close();
    });
});
```

Read a file as buffer in sync way
```javascript
zip.on('ready', () => {
    const data = zip.entryDataSync('path/inside/zip.txt');
    zip.close();
});
```

When extracting a folder, you can listen to `extract` event
```javascript
zip.on('extract', (entry, file) => {
    console.log(`Extracted ${entry.name} to ${file}`);
});
```

`entry` event is generated for every entry during loading
```javascript
zip.on('entry', entry => {
    // you can already stream this entry,
    // without waiting until all entry descriptions are read (suitable for very large archives)
    console.log(`Read entry ${entry.name}`);
});
```

## Options

You can pass these options to the constructor
- `storeEntries: true` - you will be able to work with entries inside zip archive, otherwise the only way to access them is `entry` event
- `skipEntryNameValidation: true` - by default, entry name is checked for malicious characters, like `../` or `c:\123`, pass this flag to disable validation errors
- `nameEncoding: 'utf8'` - encoding used to decode file names, UTF8 by default

## Methods

- `zip.entries()` - get all entries description
- `zip.entry(name)` - get entry description by name
- `zip.stream(entry, function(err, stm) { })` - get entry data reader stream
- `zip.entryDataSync(entry)` - get entry data in sync way
- `zip.close()` - cleanup after all entries have been read, streamed, extracted, and you don't need the archive

## Building

The project doesn't require building. To run unit tests with [nodeunit](https://github.com/caolan/nodeunit):  
```sh
npm test
```

## Known issues

- [utf8](https://github.com/rubyzip/rubyzip/wiki/Files-with-non-ascii-filenames) file names

## Out of scope

- AES encrypted files: the library will throw an error if you try to open it

## Contributors

ZIP parsing code has been partially forked from [cthackers/adm-zip](https://github.com/cthackers/adm-zip) (MIT license).
