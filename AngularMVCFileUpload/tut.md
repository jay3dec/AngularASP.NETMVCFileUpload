File upload is a common thing that we encounter when building any web application. While I was trying to build a web application using ASP>NET MVC 4 and AngularJS, I had a requirement to implement file upload. I stumbled across a couple of AngularJS directives for File upload. I choose one of the directives and here in this tutorial I'll demonstrate how to implement file upload using AngularJS and ASP.NET MVC 4.

In this tutorial, we'll make use of the [Angular Upload](https://github.com/leon/angular-upload) directive. As per the GitHub intro page, the directive is lightweight, supports all browsers and has no dependency on jQUery.

## Setting Up MVC 4 Web Project For File Upload

Let's get started by creating a ASP.NET MVC 4 web project. 
Create an empty MVC 4 web project. Once the project structure is created, right click on *controller* folder and create a new controller called *HomeController*. Here is the *HomeController* : 
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Mvc;

namespace AngularFileUploadDemo.Controllers
{
    public class HomeController : Controller
    {
        public ActionResult Index()
        {
            return View();
        }
    }
}
```
Right click on the Index method and add a view called Index. 
Inside the HomeController add a new method called *Upload* which would handle the upload functionality when the file is posted using AngularJS from the client side. Here is how the it would look like :
```
using System;
using System.Collections.Generic;
using System.Linq;
using System.Web;
using System.Web.Mvc;

namespace AngularFileUploadDemo.Controllers
{
    public class HomeController : Controller
    {
        /// <summary>
        /// Default Action Method to Render the Home View
        /// </summary>
        /// <returns></returns>
        public ActionResult Index()
        {
            return View();
        }

        /// <summary>
        /// Action Method to Handle the Upload Functionalty
        /// </summary>
        /// <param name="aFile"></param>
        [HttpPost]
        public void Upload(System.Web.HttpPostedFileBase aFile)
        {
            string file = aFile.FileName;
            string path = Server.MapPath("../Upload//");
            aFile.SaveAs(path + Guid.NewGuid() + "." + file.Split('.')[1]);
        }
    }
}
```

In the *Upload* method the posted file is accessible using the *aFile* parameter. We simply read the file name and then save the posted file using the *SaveAs* method of *System.Web.HttpPostedFileBase*.


## Using AngularJS File Upload in ASP.NET MVC 4

First download and include AngularJS into your project. Include it in the index.cshtml view.
```
<script src="../../Script/angular.js" type="text/javascript"></script>
```
Download and unzip the [angular upload](https://github.com/leon/angular-upload) project into your MVC 4 web application. Include the 
JavaScript file *angular-upload.js* in the *index* view.
```
<script src="../../Script/angular-upload.js" type="text/javascript"></script>
```

Once you have included the required files, start by creating the angular js app and inject the *lr.upload* module to the
AngularJS application.
```
var app = angular.module('myApp', ['lr.upload']);

app.controller('HomeCtrl', ['$scope', function($scope) {


}]);
```

Add the following HTML code to the *Index* view.
```
<div ng-app="myApp">
    <div ng-controller="HomeCtrl">
        <input name="myFile" upload-file="myFile" type="file" />
        <input type="button" ng-click="doUpload()" text="Upload" />
    </div>
</div>
```

Next, create a *Upload* function in the AngularJS *HomeCtrl* which would be called to upload the file to the server.
Inject the *upload* module to the *HomeCtrl* and call it to send the file to the server.
```
app.controller('HomeCtrl', ['$scope', 'upload', '$http', function($scope, upload, $http) {

    $scope.doUpload = function() {

        upload({
            url: 'Home/upload',
            method: 'POST'
        }).then(
            function(response) {
                console.log(response.data);
            },
            function(response) {
                console.error(response);
            }
        );

    }
}])
```
The above created *doupload* function does the upload functionality using the AngularJS upload directive. But until now 
we haven't passed the uploaded file to the doupload.
*ngModel* doesn't really work for input type file. So, we need to create a custom directive to get the file uploaded using
input type file which would also get updated each time the uploaded file is changed. So, let's create a directive
which can be used as an attribute along with the input type file.

```
 .directive('uploadFile', ['$parse', function($parse) {
     return {
         restrict: 'A'
     };
 }]);
```
Now the magic work would be written inside the link function of the directive. We would make use of the [$parse](https://docs.angularjs.org/api/ng/service/$parse) AngularJS
function to convert the AngularJS expression into a function and to re assign the changed input file value.
Here is how the directive code looks like:
```
 .directive('uploadFile', ['$parse', function ($parse) {
          return {
              restrict: 'A',
              link: function (scope, element, attrs) {

                  var file_uploaded = $parse(attrs.uploadFile);

                  element.bind('change', function () {
                      scope.$apply(function () {
                          file_uploaded.assign(scope, element[0].files[0]);
                      });
                  });
              }
          };
      } ]);
```

Now try running your file upload web application and it should be working fine.

## Wrapping It Up
In this tutorial we saw how implement the file upload functionality using the AngularJS and ASP.NET MVC 4.
Source code from this tutorial is available on GitHub. Do let us know your thoughts, issues or any suggestions 
in the comments below.
