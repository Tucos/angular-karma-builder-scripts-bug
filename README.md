> [!NOTE]
> This has been fixed as part of `@angular/cli` version [2.0.3](https://github.com/angular/angular-cli/releases/tag/20.0.3)

# `@angular/build:karma` with scripts configured

This has been reported as https://github.com/angular/angular/issues/62044

## Background

This project demonstrates a bug in `@angular/build:karma`.
The application depends on some global data from another script.
This script is loaded in `index.html`, with a default provided as asset.

In the application, this works fine.
`npx ng serve` will show "Global data: some data" upon browsing to the app.

In tests, this fails when using the new builder.

## Reproduction
Demonstrate that `@angular-devkit/build-angular:karma` works as expected.
```
git checkout karma-old
npm i
npx ng test --no-watch
```

The test will output:
```
✔ Browser application bundle generation complete.
13 06 2025 12:55:39.057:INFO [karma-server]: Karma v6.4.4 server started at http://localhost:9876/
13 06 2025 12:55:39.059:INFO [launcher]: Launching browsers Chrome with concurrency unlimited
13 06 2025 12:55:39.062:INFO [launcher]: Starting browser Chrome
13 06 2025 12:55:40.073:INFO [Chrome 137.0.0.0 (Windows 10)]: Connected on socket jrT_n3WRND50c02WAAAB with id 85746744
Chrome 137.0.0.0 (Windows 10): Executed 2 of 2 SUCCESS (0.399 secs / 0.047 secs)
TOTAL: 2 SUCCESS
```

Switch to the other branch, which is configured to use `@angular/build:karma`.
```
git checkout karma-new
npx ng test --no-watch
```

The test will output:
```
Initial chunk files  | Names             |  Raw size
chunk-4QII3H6J.js    | -                 |   2.15 MB | 
polyfills.js         | polyfills         |   1.01 MB | 
test_main.js         | test_main         | 237.57 kB | 
jasmine-cleanup-1.js | jasmine-cleanup-1 |  67.20 kB | 
spec-app.spec.js     | spec-app.spec     |   2.44 kB | 
chunk-TTULUY32.js    | -                 |   1.99 kB | 
jasmine-cleanup-0.js | jasmine-cleanup-0 | 519 bytes | 
styles.css           | styles            |  95 bytes | 
scripts.js           | scripts           |  86 bytes | 

                     | Initial total     |   3.47 MB

Application bundle generation complete. [1.629 seconds]

13 06 2025 13:02:20.188:INFO [karma-server]: Karma v6.4.4 server started at http://localhost:9876/
13 06 2025 13:02:20.190:INFO [launcher]: Launching browsers Chrome with concurrency unlimited
13 06 2025 13:02:20.196:INFO [launcher]: Starting browser Chrome
13 06 2025 13:02:21.207:INFO [Chrome 137.0.0.0 (Windows 10)]: Connected on socket 3VDA0IkNDg1AuL4_AAAB with id 94725245
Chrome 137.0.0.0 (Windows 10) App should show global data FAILED
        ReferenceError: someglobalobject is not defined
            at App2.get data [as data] (src/app/app.ts:10:5)
            at App2_Template (ng:///App2.js:12:55)
            at executeTemplate (node_modules/@angular/core/fesm2022/debug_node-JnOYh9kg.mjs:7140:9)
            at refreshView (node_modules/@angular/core/fesm2022/debug_node-JnOYh9kg.mjs:8888:13)
            at detectChangesInView (node_modules/@angular/core/fesm2022/debug_node-JnOYh9kg.mjs:9108:9)
            at detectChangesInViewIfAttached (node_modules/@angular/core/fesm2022/debug_node-JnOYh9kg.mjs:9068:5)
            at detectChangesInComponent (node_modules/@angular/core/fesm2022/debug_node-JnOYh9kg.mjs:9056:5)
            at detectChangesInChildComponents (node_modules/@angular/core/fesm2022/debug_node-JnOYh9kg.mjs:9134:9)
            at refreshView (node_modules/@angular/core/fesm2022/debug_node-JnOYh9kg.mjs:8943:13)
            at detectChangesInView (node_modules/@angular/core/fesm2022/debug_node-JnOYh9kg.mjs:9108:9)
Chrome 137.0.0.0 (Windows 10): Executed 2 of 2 (1 FAILED) (0.283 secs / 0.277 secs)                                                                                                                                                                                   
TOTAL: 1 FAILED, 1 SUCCESS
```

## Structure / configuration

The global data has a declaration in `src/app/someglobalobject.ts`.
The default is provided by `src/someglobalobject.js`, which is included in `index.html`.

For the test to work, the script must be included.
This is configured in `angular.json`:
```json
{
  "test": {
    "builder": "@angular/build:karma",
    "options": {
      "assets": [
        "src/someglobalobject.js"
      ],
      "scripts": [
        "src/someglobalobject.js"
      ]
    }
  }
}
```

 The asset is served, but the script is not included in the html.
