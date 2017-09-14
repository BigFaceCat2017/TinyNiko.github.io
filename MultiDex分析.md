# DexClassLoader 分析
尝试了一下oneNote的导出功能，貌似并不是很好，所以只能改用markdown了。 
muLtidex 的加载，实在attachBaseContext的时候调用Multidex.install(this) 就可以了Install 实现比较简单

```java
158                File dexDir = new File(applicationInfo.dataDir, SECONDARY_FOLDER_NAME);
159                List<File> files = MultiDexExtractor.load(context, applicationInfo, dexDir, false);
160                if (checkValidZipFiles(files)) {
161                    installSecondaryDexes(loader, dexDir, files);
162                } else {
163                    Log.w(TAG, "Files were not valid zip files.  Forcing a reload.");
164                    // Try again, but this time force a reload of the zip file.
165                    files = MultiDexExtractor.load(context, applicationInfo, dexDir, true);
166
167                    if (checkValidZipFiles(files)) {
168                        installSecondaryDexes(loader, dexDir, files);
169                    } else {
170                        // Second time didn't work, give up
171                        throw new RuntimeException("Zip files were not valid.");
172                    }
173                }

```


最重要的是MultiDexExtractor.load 调用了两次，然后调用installSecondaryDexes 进行安装
```java
 
   */
82    static List<File> load(Context context, ApplicationInfo applicationInfo, File dexDir,
83            boolean forceReload) throws IOException {
84        Log.i(TAG, "MultiDexExtractor.load(" + applicationInfo.sourceDir + ", " + forceReload + ")");
85        final File sourceApk = new File(applicationInfo.sourceDir);
86
87        long currentCrc = getZipCrc(sourceApk);
88
89        List<File> files;
90        if (!forceReload && !isModified(context, sourceApk, currentCrc)) {
91            try {
92                files = loadExistingExtractions(context, sourceApk, dexDir);
93            } catch (IOException ioe) {
94                Log.w(TAG, "Failed to reload existing extracted secondary dex files,"
95                        + " falling back to fresh extraction", ioe);
96                files = performExtractions(sourceApk, dexDir);
97                putStoredApkInfo(context, getTimeStamp(sourceApk), currentCrc, files.size() + 1);
98
99            }
100        } else {
101            Log.i(TAG, "Detected that extraction must be performed.");
102            files = performExtractions(sourceApk, dexDir);
103            putStoredApkInfo(context, getTimeStamp(sourceApk), currentCrc, files.size() + 1);
104        }
105
106        Log.i(TAG, "load found " + files.size() + " secondary dex files");
107        return files;
108    }
 

```

首先是一个判断，看是不是强制，还有就是有没有修改过apk，没有的话就直接用已经存在的，loadExistingExtractions 找了zip文件进行加载
Perform 做的事就是把apk里的其他dex文件放到一个zip文件中，这样，以后加载就直接加载zip文件，看了其他版本的代码，加载的流程有些不一样，不过并不影响

```java
179            ZipEntry dexFile = apk.getEntry(DEX_PREFIX + secondaryNumber + DEX_SUFFIX);
180            while (dexFile != null) {
181                String fileName = extractedFilePrefix + secondaryNumber + EXTRACTED_SUFFIX;
182                File extractedFile = new File(dexDir, fileName);
183                files.add(extractedFile);
184
185                Log.i(TAG, "Extraction is needed for file " + extractedFile);
186                int numAttempts = 0;
187                boolean isExtractionSuccessful = false;
188                while (numAttempts < MAX_EXTRACT_ATTEMPTS && !isExtractionSuccessful) {
189                    numAttempts++;
190
191                    // Create a zip file (extractedFile) containing only the secondary dex file
192                    // (dexFile) from the apk.
193                    extract(apk, dexFile, extractedFile, extractedFilePrefix);
194
195                    // Verify that the extracted file is indeed a zip file.
196                    isExtractionSuccessful = verifyZipFile(extractedFile);
197
198                    // Log the sha1 of the extracted zip file
199                    Log.i(TAG, "Extraction " + (isExtractionSuccessful ? "success" : "failed") +
200                            " - length " + extractedFile.getAbsolutePath() + ": " +
201                            extractedFile.length());
202                    if (!isExtractionSuccessful) {
203                        // Delete the extracted file
204                        extractedFile.delete();
205                        if (extractedFile.exists()) {
206                            Log.w(TAG, "Failed to delete corrupted secondary dex '" +
207                                    extractedFile.getPath() + "'");
208                        }
209                    }
210                }
211                if (!isExtractionSuccessful) {
212                    throw new IOException("Could not create zip file " +
213                            extractedFile.getAbsolutePath() + " for secondary dex (" +
214                            secondaryNumber + ")");
215                }
216                secondaryNumber++;
217                dexFile = apk.getEntry(DEX_PREFIX + secondaryNumber + DEX_SUFFIX);

```

接着把文件相关信息写到一个shared_pref文件中
 
完事以后呢，就调用 installSecondaryDexes 进行正式的安装
```java
237    private static void installSecondaryDexes(ClassLoader loader, File dexDir, List<File> files)
238            throws IllegalArgumentException, IllegalAccessException, NoSuchFieldException,
239            InvocationTargetException, NoSuchMethodException, IOException {
240        if (!files.isEmpty()) {
241            if (Build.VERSION.SDK_INT >= 19) {
242                V19.install(loader, files, dexDir);
243            } else if (Build.VERSION.SDK_INT >= 14) {
244                V14.install(loader, files, dexDir);
245            } else {
246                V4.install(loader, files);
247            }
248        }
249    }
250

```

根据不同版本进行不同的安装
从代码上看，其实都是去反射调用了DexPathList ,然后再调用这个对象的makeDexElements ， 看到这个makeDexELements，基本上就知道要完事了，这个就和DexClassLoader加载过程差不多了。用来做不落地加载好像也是可以的
```java
417        private static Object[] makeDexElements(
418                Object dexPathList, ArrayList<File> files, File optimizedDirectory,
419                ArrayList<IOException> suppressedExceptions)
420                        throws IllegalAccessException, InvocationTargetException,
421                        NoSuchMethodException {
422            Method makeDexElements =
423                    findMethod(dexPathList, "makeDexElements", ArrayList.class, File.class,
424                            ArrayList.class);
425
426            return (Object[]) makeDexElements.invoke(dexPathList, files, optimizedDirectory,
427                    suppressedExceptions);
428        }
429    }

```



```java
328    private static void expandFieldArray(Object instance, String fieldName,
329            Object[] extraElements) throws NoSuchFieldException, IllegalArgumentException,
330            IllegalAccessException {
331        Field jlrField = findField(instance, fieldName);
332        Object[] original = (Object[]) jlrField.get(instance);
333        Object[] combined = (Object[]) Array.newInstance(
334                original.getClass().getComponentType(), original.length + extraElements.length);
335        System.arraycopy(original, 0, combined, 0, original.length);
336        System.arraycopy(extraElements, 0, combined, original.length, extraElements.length);
337        jlrField.set(instance, combined);
338    }
339

```
这个expand也是个反射调用，把优化后的dex文件加到数组里


分析到这里基本就结束了，其实主要就是把multidex过一遍