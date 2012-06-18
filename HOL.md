#Using IndexedDB#

## Overview ##

HTML5 is the emerging standard for building and writing HTML webpages for web applications and sites. One of the things that HTML5 is trying to solve is the problem of storing data on the client-side. In the past, storage mechanisms on the client-side were very limited. Today, with the emerging HTML5 JavaScript APIs there are new storage capabilities such as Web Storage API and IndexedDB API. 
The HTML5 IndexedDB API provides a client-side index database that includes querying and storage mechanisms. The IndexedDB storage can save JavaScript objects and exposes the options to retrieve them very fast.  
In this lab, you will use the HTML5 IndexedDB API in order to save the images that are being created in the FacePlace application.

### Objectives ###

In this hands-on lab, you will learn how to:

-	Use the new HTML5 IndexedDB API in order to save the images in the FacePlace application.

 
### Prerequisites ###

-	Internet Explorer 10
-	An HTML editor of your choice
-	Prior knowledge of HTML and JavaScript development 
-	Http context using IIS or IIS Express. Refer to “New HTML5 Features” Appendix Section for more information.

## Exercises ##

This hands-on lab includes the following exercises:

1. [Exercise 1: Using IndexedDB for storing images](#Exercise1)

Estimated time to complete this lab: **30-45 minutes**.

<a name="Exercise1" />
### Exercise 1 - Using IndexedDB for Storing Images ###

The starting point for this exercise is the solution located in the lab installation folder under the **Source\Begin** folder.
The solution contains a project with all the storage functionality implemented. As you progress through the exercise, you will gradually create the **database** object and its integration into the **FacePlace** application.

<a name="Ex1Task1" />
#### Task 1 - Creating the Database JavaScript Object Using IndexedDB API ####

In this task, we will create a JavaScript object by the name **database** that will handle the image storage functionality using the IndexedDB API. The **database** object will expose functions to query, add and remove images.

1.	Open the solution under the **Source\Begin** folder and examine the project.
1.	Open the **Face.Database.js** file, which is located in the **Scripts** folder.
1.	Declare the immediate function that will create the scope for the **database**.

	````JavaScript
	(function ()
	{
		"use strict";
	}());
	````

	> **Note:** It is a good practice to use JavaScript immediate functions when declaring JavaScript object. Immediate functions do not pollute the global JavaScript scope. 

1.	Create the **database** object inside the immediate function and set it to the face namespace after its creation if it does not exists.
	
	<!-- mark:4-11 -->
	````JavaScript
	(function ()
	{
		"use strict";
		if (!face.database)
		{
			 var database = {
					
			 };

			 face.database = database;
		}
	}());
	````

1.	Inside the **database** object, create three variables that will help to support the database object across some of the major browsers.  
	
	<!-- mark:7-11 -->
	````JavaScript
	(function ()
	{
		"use strict";
		if (!face.database)
		{
			 var database = {
				//IndexedDB system objects, support cross browsing
				indexedDB: window.indexedDB || window.webkitIndexedDB || window.mozIndexedDB || 	
						   window.msIndexedDB,
				IDBTransaction: window.IDBTransaction || window.webkitIDBTransaction,
				IDBKeyRange: window.IDBKeyRange || window.webkitIDBKeyRange,
			 };

			 face.database = database;
		}
	}());
	````

	> **Note:** Since the IndexedDB API is an unstable HTML5 specification, each browser vendor creates its API implementation with the browser prefix (in Internet Explorer 10 it is the ms prefix). When the API will become stable enough the vendors will remove their prefixes. 

1.	Add the indexedDB settings variables - the database name, the **objectStore** names and the database version. Add the code below the IndexedDB Settings comment inside the **database** object, right after the lines you added in the previous step. 

	<!-- mark:7-12 -->
	````JavaScript
	...
	var database = {
		//IndexedDB system objects, support cross browsing
		indexedDB: window.indexedDB || window.webkitIndexedDB || window.mozIndexedDB || window.msIndexedDB,
		IDBTransaction: window.IDBTransaction || window.webkitIDBTransaction,
		IDBKeyRange: window.IDBKeyRange || window.webkitIDBKeyRange,
		//IndexedDB Settings
		dbName: "TheFacePlace",
		tables: {
			 faces: { tableName: "faces", imageName: "imageName", imageData: "imageData" }
		},
		dbVersion: 1,
	};
	...
	````

1.	Add the last three variables that hold the database object, indicate whether the database was initialized and hold the open handlers. Add the code below the IDBDatabase object comment at the end of the **database** object.

	<!-- mark:10-13 -->
	````JavaScript
	...
	var database = {
		...
		//IndexedDB Settings
		dbName: "TheFacePlace",
		tables: {
			 faces: { tableName: "faces", imageName: "imageName", imageData: "imageData" }
		},
		dbVersion: 1,
		//IDBDatabase object
		db: null,
		isDbInitialized: false,
		onopenedHandlers: [],
	};
	...
	````

1.	Add the **init** function and the **open** function. The **init** function will call the **open** function. The **open** function is responsible to open the IndexedDB database in order to enable interaction with the IndexedDB. The **open** function is also responsible to create the faces **objectStore**, which is used to store all the images. Add these functions inside the **database** objects, right after the code you added in the previous step.
	
	<!-- mark:5-46 -->
	````JavaScript
	...
	var database = {
		...
		onopenedHandlers: [],
		init: function()
		{
			 this.open();
		},
		open: function()
		{
			 if(!face.isIndexedDbSupported) {
				  alert('Your browser does not support IndexedDB API');
				  return;
			 }
		
			 var isNewDb = false,
				  request = this.indexedDB.open(this.dbName, database.dbVersion);
			 request.onerror = database.onerror;
			 request.onsuccess = function()
			 {
				  database.db = request.result;
		
				  database.isDbInitialized = true;
		
				  //Add Samples
				  if (isNewDb) {
						database.addSampleItems();
				  }
		
				  //Call to handlers callbacks
				  database.callHandlers(database.onopenedHandlers);
			 };
			 request.onupgradeneeded = function(e)
			 {
				  isNewDb = true;
		
				  // Create a 'store' to store the images.
				  // We're going to use "imageName" as our key path because it's guaranteed to be unique.
				  var store = e.currentTarget.result.createObjectStore(database.tables.faces.tableName
									, { keyPath: database.tables.faces.imageName, autoIncrement: true });
		
				  // Create an index to search customers by name
				  store.createIndex(database.tables.faces.imageName, database.tables.faces.imageName
									, { unique: true });
			 };
		},
	};
	...
	````

1.	Add three utility functions - **onopend**, **callHandlers** and **checkDbInitialize**. The first function will call a callback function on the database object if it is opened. The second function will call all the event handlers that it receives. The third function will check whether the database was initialized. Add these functions inside the **database** objects, right after the code you added in the previous step.

	<!-- mark:8-42 -->
	````JavaScript
	...
	var database = {
		...
		open: function()
		{
			 ...
		},
		onopened: function(callback)
		{
			 if (database.isDbInitialized)
			 {
				  if (callback !== undefined)
				  {
						callback.call(this);
				  }
			 } else
			 {
				  database.onopenedHandlers.push(callback);
			 }
		},
		
		callHandlers: function(handlers)
		{
			 if (handlers !== undefined)
			 {
				  var handler;
				  while ((handler = handlers.pop()) !== undefined)
				  {
						handler();
				  }
			 }
		},
		
		checkDbInitialize: function()
		{
			 if (!this.isDbInitialized)
			 {
				  return false;
			 }
		
			 return true;
		},
	};
	...
	````

1. Add the **onerror** function. The **onerror** function is the default function to handle errors. It is writing to the console its function arguments. Add this function inside the **database** object, below the **checkDbInitialize** function.

	<!-- mark:8-14 -->
	````JavaScript
	...
	var database = {
		...
		checkDbInitialize: function()
		{
			 ...
		},
		onerror: function()
		{
			 if (window.console !== undefined)
			 {
				  window.console.log.apply(window.console, arguments);
			 }
		},
	};
	...
	````

1.	Add the **addItem** function. The **addItem** function will create a transaction that will add the supplied image to the faces **objectStore**. In addition, another function named **addSampleItems** is added, which will add an image sample by default. Add these functions inside the **database** object, right after the code you added in the previous step.
	
	<!-- mark:8-59 -->
	````JavaScript
	...
	var database = {
		...
		onerror: function()
		{
			 ...
		},
		addSampleItems: function() {
			 var i;
			 
			 //Add Javascript Dynamically
			 $('<script src="Scripts/Face.Database.SampleImages.js"></script>').appendTo('head');
		
			 if (face.imageSamples !== undefined) {
				  for (i in face.imageSamples) {
						this.addItem(i, face.imageSamples[i].data);
				  }
			 }
		},
		
		addItem: function(imageName, imageData, callbackOnSuccess, callbackOnError) {
			 var transaction, objectStore, imageObject, request;
			 
			 if (!this.checkDbInitialize()) {
				  return null;
			 }
		
			 //Open a new transaction
			 transaction = this.db.transaction(this.tables.faces.tableName, this.IDBTransaction.READ_WRITE);
			 transaction.onerror = database.onerror;
		
			 //Get the object (table) to store the data in.
			 objectStore = transaction.objectStore(this.tables.faces.tableName);
		
			 //Store the image
			 try {
				  imageObject = { };
				  imageObject[database.tables.faces.imageName] = imageName;
				  imageObject[database.tables.faces.imageData] = imageData;
		
				  request = objectStore.add(imageObject);
				  request.onsuccess = function(evt) {
						if (callbackOnSuccess !== undefined && callbackOnSuccess !== null) {
							 callbackOnSuccess.call(this, evt);
						}
				  };
				  request.onerror = function(evt) {
						database.onerror("addItem.request.onerror", evt);
		
						if (callbackOnError !== undefined && callbackOnError !== null) {
							 callbackOnError.call(this, evt);
						}
				  };
			 } catch(e) {
				  database.onerror("addItem.catch", e);
			 }
		
			 return this;
		},
	};
	...
	````

1.	Add the **getItem** function. The **getItem** function will retrieve an image, which is stored in the images **objectStore** with the provided **imageName** index. Then, the **getItem** function will call the callback function that was supplied as a second parameter. Add this function inside the **database** object, right after the code you added in the previous step.
	
	<!-- mark:7-31 -->
	````JavaScript
	...
	var database = {
		...
		addItem: function(imageName, imageData, callbackOnSuccess, callbackOnError) {
			 ...
		},
		getItem: function(imageName, callback)
		{
			 var trans, store, request;
		
			 if (!this.checkDbInitialize())
			 {
				  return null;
			 }
		
			 trans = this.db.transaction(this.tables.faces.tableName);
			 store = trans.objectStore(this.tables.faces.tableName);
		
			 // Get the data by 'Image Name'
			 request = store.get(imageName);
			 request.onerror = database.onerror;
			 request.onsuccess = function(event)
			 {
				  if (callback !== undefined && event.target.result !== undefined)
				  {
						callback.call(this, event.target.result.imageData);
				  }
			 };
		
			 return this;
		},
	};
	...
	````

1. Add the **getItems** function. The **getItems** function will open a cursor on the faces **objectStore** and will retrieve all the items that exists in a range between the supplied from parameter and the from+count. Add this function inside the **database** object, right after the code you added in the previous step.
	
	<!-- mark:8-40 -->
	````JavaScript
	...
	var database = {
		...
		getItem: function(imageName, callback)
		{
			 ...
		},
		getItems: function(from, count, callback) {
			var current, trans, store, request, inRange;
			if (!this.checkDbInitialize()) {
			  return null;
			}

			current = 0;

			//Open a new transaction
			trans = this.db.transaction(this.tables.faces.tableName);
			trans.onerror = database.onerror;

			//Get the object (table) to store the data in.
			store = trans.objectStore(this.tables.faces.tableName);

			request = store.index(database.tables.faces.imageName).openCursor();
			request.onerror = database.onerror;
			request.onsuccess = function(event) {
			  var cursor = event.target.result || event.result || request.result;
			  if (cursor) {
					inRange = from <= current && current < from + count;
					if (inRange) {
						 if (callback !== undefined && cursor.value !== undefined) {
							  callback.call(this, cursor.value.imageName, cursor.value.imageData);
						 }
					}
					current += 1;
					cursor.continue();
			  }
			};

			return this;
		},
	};
	...
	````

1.	Add the **getItemCount** function. The **getItemCount** function returns the number of items in the faces **objectStore**. Add this function inside the **database** object, right after the code you added in the previous step.
	
	<!-- mark:7-32 -->
	````JavaScript
	...
	var database = {
		...
		getItems: function(from, count, callback) {
			...	
		},
		getItemCount: function(callback)
		{
			 var trans, store, request, count;
		
			 if (!this.checkDbInitialize())
			 {
				  return null;
			 }
		
			 trans = this.db.transaction(this.tables.faces.tableName, this.IDBTransaction.READ_WRITE);
			 store = trans.objectStore(this.tables.faces.tableName);
		
			 request = store.count();
			 request.onerror = database.onerror;
			 request.onsuccess = function(event)
			 {
				  count = event.target.result;
		
				  if (callback !== undefined && count !== undefined)
				  {
						callback.call(this, count);
				  }
			 };
		
			 return this;
		},
	};
	...
	````

1.	Add the **deleteItem** function. The **deleteItem** function deletes the item, which relates to the key parameter that was provided to the function. Add this function inside the **database** object, right after the code you added in the previous step.
	
	<!-- mark:8-31 -->
	````JavaScript
	...
	var database = {
		...
		getItemCount: function(callback)
		{
			 ...
		},
		deleteItem: function(key, callback)
		{
			 var trans, store, request;
		
			 if (!this.checkDbInitialize())
			 {
				  return null;
			 }
		
			 trans = this.db.transaction(this.tables.faces.tableName, this.IDBTransaction.READ_WRITE);
			 store = trans.objectStore(this.tables.faces.tableName);
			 request = store.delete(key);
		
			 request.onerror = database.onerror;
			 request.onsuccess = function()
			 {
				  if (callback !== undefined)
				  {
						callback();
				  }
			 };
		
			 return this;
		}
	};
	...
	````

1.	Add the **renameItem** function. The **renameItem** function renames the item already stored in the database. Add this function inside the **database** object, right after the code you added in the previous step.
	
	<!-- mark:8-36 -->
	````JavaScript
	...
	var database = {
		...
		deleteItem: function(key, callback)
		{
			...
		},
		renameItem: function(oldName, newName, callback) {
			var trans, store, getRequest, record, setRequest;

			if (!this.checkDbInitialize()) {
			  return null;
			}

			trans = this.db.transaction(database.tables.faces.tableName, this.IDBTransaction.READ_WRITE);
			store = trans.objectStore(database.tables.faces.tableName);

			// Get the data by 'Image Name'
			getRequest = store.get(oldName);
			getRequest.onerror = database.onerror;
			getRequest.onsuccess = function(event) {
			  if (event.target.result !== undefined) {
					record = event.target.result;
					record.imageData = null;
					setRequest = store.put(record, oldName);
					setRequest.onerror = database.onerror;
					setRequest.onsuccess = function() {
						 if (callback !== undefined && event.target.result !== undefined) {
							  callback.call(this);
						 }
					};
			  }
			};

			return this;
		},
	};
	...
	````

1.	Add the database initialization after the **database** object declaration and before you set it to the face namespace. 
	
	<!-- mark:11-12 -->
	````JavaScript
	...
	(function ()
	{
		"use strict";
		if (!face.database)
		{
			var database = {
				
			};

			//Initialize the IndexedDB
			database.init();

			face.database = database;
		}
	}());
	````

<a name="Ex1Task2" />
#### Task 2 - Creating the ImageManager JavaScript Object ####

In this task, we will create the **ImageManager** JavaScript object, which is responsible to interact with the **database** JavaScript object that was created in the previous task. The **ImageManager** is also responsible to draw images into the image canvas. 

1.	Open the **Face.ImageManager.js** file, which is located in the **Scripts** folder.
1.	Declare the immediate function that will create the scope for the **imageManager**.

	````JavaScript
	(function ()
	{
		"use strict";
	}());
	````

	> **Note:** It is a good practice to use JavaScript immediate functions when declaring JavaScript object. Immediate functions do not pollute the global JavaScript scope. 

1.	Create the **ImageManager** object inside the immediate function and set it to the face namespace after its creation if it does not exists.
	
	<!-- mark:4-11 -->
	````JavaScript
	(function ()
	{
		"use strict";
		if (!face.imageManager)
		{
			var ImageManager = function (database) {

			};

			face.imageManager = new ImageManager(face.database);
		}
	}());
	````
1.	Inside the **ImageManager** object, add two properties with the names **that** and **_database**. The first property will hold the instance of the **ImageManager** and the second property will hold the database object. 

	<!-- mark:7-8 -->
	````JavaScript
	(function ()
	{
		"use strict";
		if (!face.imageManager)
		{
			var ImageManager = function (database) {
				this.that = this;
				this._database = database;
			};

			face.imageManager = new ImageManager(face.database);
		}
	}());
	````

1.	Add the **insertImage** function inside the **ImageHandler** object. The **insertImage** function gets a **name**, which will be the image key in the storage, an **imageSource** that is the source for the image and a **callback** function to handle the callbacks. The function will load the image into an **Image** object and then set it into to webpage’s canvas and save it into the database object. 
	
	<!-- mark:6-41 -->
	````JavaScript
	...
	var ImageManager = function (database) {
		this.that = this;
		this._database = database;
		
		this.insertImage = function (name, imageSource, callback) {
			var image = new Image();
			var that = this;
			image.onload = function () {
				var canvas = document.createElement("canvas");
				canvas.width = image.width;
				canvas.height = image.height;

				var context = canvas.getContext("2d");
				context.drawImage(image, 0, 0);

				var url = canvas.toDataURL();
				that._database.addItem(name, url, function () {
					if (callback !== undefined) {
						 callback(true);
					}
				}, function () {
					if (callback !== undefined) {
						 callback(false);
					}
				});
			};

			image.src = imageSource;
			if (imageSource === null || imageSource === undefined) {
				that._database.addItem(name, null, function () {
					if (callback !== undefined) {
						 callback(true);
					}
				}, function () {
					if (callback !== undefined) {
						 callback(false);
					}
				});
			}
		};
	};
	...
	````

1.	Add the **deleteImage** function inside the **ImageHandler** object. The **deleteImage** function is responsible to delete the image that is related to its provided name key. 
	
	<!-- mark:7-13 -->
	````JavaScript
	...
	var ImageManager = function (database) {
		...
		this.insertImage = function (name, imageSource, callback) {
			...
		};
		this.deleteImage = function (name, callback) {
			this._database.deleteItem(name, function () {
				if (callback !== undefined) {
					callback();
				}
			});
		};
	};
	...
	````

1.	Add the **retrieveImage** function inside the **ImageHandler** object. The **retrieveImage** function gets the name of the image to retrieve and retrieve it from the database object. 
	
	<!-- mark:7-15 -->
	````JavaScript
	...
	var ImageManager = function (database) {
		...
		this.deleteImage = function (name, callback) {
			...
		};
		this.retrieveImage = function (name, callback) {
			this._database.getItem(name, function (imageData) {
				var image = new Image();
				image.src = imageData;
				if (callback !== undefined) {
					callback(image);
				}
			});
		};
	};
	...
	````

1.	Add the **renameImage** function inside the **ImageHandler** object. The **renameImage** function will replace a stored image’s key with a new name. It will do it by first retrieving the item by its old name, add the retrieved item to the database with the new name and then delete the item, which is being referenced by the old name. 
	
	<!-- mark:7-22 -->
	````JavaScript
	...
	var ImageManager = function (database) {
		...
		this.retrieveImage = function (name, callback) {
			...
		};
		this.renameImage = function (oldName, newName, callback) {
			var db = this._database;
			if (oldName !== newName) {
			  db.getItem(oldName, function (imageData) {
					db.addItem(newName, imageData, function () {
						 db.deleteItem(oldName, function () {
							  if (callback !== undefined) {
									callback();
							  }
						 });
					}, function () {
						 return false;
					});
			  });
			}
		};
	};
	...
	````

1.	Add the **getImageCount** function inside the **ImageHandler** object. The **getImageCount** function will retrieve the number of images that are stored in the database object. The image count is used by the FacePlace application in order to find out how many images are stored in the faces **objectStore**.

	<!-- mark:7-11 -->
	````JavaScript
	...
	var ImageManager = function (database) {
		...
		this.renameImage = function (oldName, newName, callback) {
			...
		};
		this.getImageCount = function (callback) {
			return this._database.getItemCount(function (count) {
				callback(count);
			});
		};
	};
	...
	````

1.	Add the **retrieveImages** function. The **retrieveImages** function gets two parameters a **from** parameter and a **count** parameter. The function will retrieve all the images between the from and the from+count range. 
	
	<!-- mark:7-16 -->
	````JavaScript
	...
	var ImageManager = function (database) {
		...
		this.getImageCount = function (callback) {
			...
		};
		this.retrieveImages = function (from, count, callback) {
			 this._database.getItems(from, count, function (imageName, imageData) {
				  var image = new Image();
				  image.src = imageData;
		
				  if (callback !== undefined) {
						callback(imageName, image);
				  }
			 });
		};
	};
	...
	````

1.	Add the **createImageAndExecuteCallback** function. The **createImageAndExecuteCallback** function is responsible to create an image that was supplied as parameter and call the supplied callback function. 
	
	<!-- mark:7-13 -->
	````JavaScript
	...
	var ImageManager = function (database) {
		...
		this.retrieveImages = function (from, count, callback) {
			 ...
		};
		this.createImageAndExecuteCallback = function (imageData, callback) {
			var image = new Image();
			image.src = imageData;
			if (callback !== undefined) {
				 callback(image);
			}
		};
	};
	...
	````

This step concludes the current task.

<a name="Ex1Task3" />
#### Task 3 - Integrating the Database JavaScript Object Into the ImageHandler ####

In this task, we will integrate the database object that was created in task 1 into the **ImageHandler** object. The **ImageHandler** exposes two functions, which interact with the database object, which are **savePic** and **getImage**. In this task, you will add those functions to the **ImageHandler** object.

1.	Open the **ImageHandler.js** file, which is located in the **Scripts** folder.
2.	After the declaration of the **toDataURL** function, add the **savePic** function as shown in the next code block. The **savePic** function receives an **imageName** and save the image, which exists in the ihkinetic object into the database object that exists in the face namespace. 
	
	<!-- mark:6-33 -->
	````JavaScript
	...
	this.toDataURL = function (callBack) {
	    ihkinetic.toDataURL(callBack);
	};
	
	/// <summary>
	///  Save Image to IndexedDB
	/// </summary>
	this.savePic = function (imageName) {
	   if (imageName !== undefined && imageName !== "") {
	      ihkinetic.toDataURL(function (strData) {
	                 face.database.addItem(imageName, strData,
	            function () {
	                alert('saved');
	                $('#btnSave').parent().attr('style', '');
	            },
	            function (evt) {
	                switch (evt.target.errorCode) {
	                    case 4:
	                        alert("Image already exists");
	                        break;
	                    default:
	                    if (window.console !== undefined) {
	                            console.log(evt.target.errorCode);
	                    }
	                }
	                $('#btnSave').parent().attr('style', '');
	            });
	      });
	   } else {
	      alert("Insert a name");
	   }
	};
	...
	````

1.	Add the **getImage** function after the **savePic** function. The **getImage** function retrieves a stored image and calls a callback function after the image was retrieved. 
	
	<!-- mark:8-14 -->
	````JavaScript
	...
	/// <summary>
	///  Save Image to IndexedDB
	/// </summary>
	this.savePic = function (imageName) {
	   ...
	};
	this.getImage = function (imageName, callback) {
	   face.database.getItem(imageName, function (data) {
	      if (callback !== undefined) {
	         callback.call(this, data);
	      }
	   });
	};
	...
	````
This step concludes the current task.

<a name="Ex1Task4" />
#### Task 4 - Integrating the Database JavaScript Object Into the ImageHandler ####

In this task, we will integrate the database object, which was created in task 1, into the NewGame and OpenGame webpages. When using IndexedDB in a webpage, there is a need to open the database in order to use it. Therefore, the code that you will add in this task will use the **onopened** API function that the database object exposes in order to perform operations after the database was open for use.  

1.	Open the **NewGame.js** file, which is located in the **Scripts** folder.
1.	Locate the **openGameFromStorage** function. Add the **onopened** call to the **face.database** like in the following code. 
	
	<!-- mark:9-14 -->
	````JavaScript
	...
	function openGameFromStorage() {
	    if (!!window.sessionStorage) {
	        var name = window.sessionStorage.getItem("gameId");
	        if (name !== undefined && name != null && name != "") {
	            $('#chooseImageSection').hide();
	            $('#kineticContainer').show();
	
	            face.database.onopened(function() {
	                imageHandler.getImage(name, function(data) {
	                    imageHandler.addBackground(data);
	                    window.sessionStorage.clear();
	                });
	            });
	        }
	    }
	}
	...
	````

1.	Open the **OpenGame.js** file, which is located in the **Scripts** folder.
1.	Inside the jQuery document ready call, add the **onopened** call to the **face.database** like in the following code. 
	
	<!-- mark:8-12 -->
	````JavaScript
	...
	$(function() {
	    if (face.isSupportedBrowser) {
	        initializeButtons();
	        initializeDropZone();
	        setHelpIconEvents();
	
	        face.database.onopened(function() {
	            face.pager.currentPage = 0;
	
	            loadImages();
	        });
	    } else {
	        alert("The File APIs are not fully supported in this browser.");
	    }
	});
	...
	````

This step concludes the current task and the current exercise.

## Summary ##

This lab has shown you how to use the new HTML5 IndexedDB JavaScript API and how to integrate it into an application. IndexedDB can store more than image data and can be used as a storage mechanism on the client-side. It opens new opportunities for creating much more powerful web applications and sites.