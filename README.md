# Desktop-Shiny-Apps-for-LAC
Using the RInno package in the LA County network can pose workflow issues. Follow these steps deploy Shiny apps on secure locations.

# The Problem
You can create a wonderful Shiny application with R that processes data and empowers others to be productive, but how do you deploy it to non-R-users? Your options are either a web-based application (hosted on a Shiny server) or a desktop version. This brief tutorial shows you how to use the RInno package to turn your Shiny app into an executable file that can be installed on any Windows PC. 

### RInno in the LA County Environment
By default, RInno creates two files that deal with making sure that your Shiny app has the packages it needs to work: `ensure.R` and `package_manager.R`. These files essentially check to see what packages are needed and uses an internet connection to install them, if not already installed. Unfortunately, we cannot expect this install process to work properly every single time because there are security protocols in place that require LA County network users to sign-in with credentials and even then the packages may fail to download or improperly install.  

# The Solution
By modifying the two files that check for and install packages, we can bypass this issue with internet connectivity and security. Also, we can bundle the packages required for the application in the executable file so that there is no need to download them from anywhere else. Make sure to use R version 3.4.3 and the latest version of RInno to create your application __files__. You may use any version to actually create your app, but RInno seems to work best with the latest version of R. 

### Step 1: Manage your library
As you build your application, make note of the packages and package version you are using. Also make note of the R version you are using. Ensure that you have identified all the dependencies that your packages may need. This can be difficult when you use many packages because one package may depend on a dozen other packages, so I suggest installing the packages you are using into a separate folder. You can accomplish this by doing something like:
  
`install.packages( "ggplot2", lib = "C:/Users/e12345/Desktop/temp_library" )`

Or if you want an older version of a package, you can find its URL and download it like this:
  
`packageurl <- "http://cran.r-project.org/src/contrib/Archive/ggplot2/ggplot2_0.9.1.tar.gz"
install.packages(packageurl, repos=NULL, type="source", lib = "C:/Users/e12345/Desktop/temp_library")`
  
Assuming you have already created this folder on your desktop called, "temp_library", this code will install the specified package and its dependencies. **Note: This is not intended to be your working library, just a separate place to manage your packages** 

You should do install your app's packages in a temporary folder in addition to the three packages that RInno itself requires to work: jsonlite, shiny, and magrittr. You also want to make sure that all the R base packages are included in your temporary folder. This would depend on which version of R you are using.

### Step 2: Create your app
  
* Install the latest `RInno` from CRAN or Github. 
* Install Inno Setup (to compile)
* Use `create_app()`
* Modify `ensure.R` and `package_manager.R`
* Compile the app

The code:

```r
# If you don't have development tools, install them
install.packages("devtools"); require(devtools)
# Use install_github to get RInno
devtools::install_github("ficonsulting/RInno",  build_vignettes = TRUE)require(RInno)
# Use RInno to get Inno Setup
RInno::install_inno()

create_app(
    app_name    = "My App Name", # this just calls it  
    app_dir     = "C:/path/to/MyApp", # location of my app
    # pkgs        = c("shiny"),  # Don't use this argument
    # remotes     = c("daattali/shinyjs"), # Don't use this argument
    user_browser = "ie",  # LA County uses Internet Explorer as the standard browser
    include_R   = TRUE,     # Download R and install it with your app
    R_version   = "3.4.3",  # Specify the version of R you want
    privilege   = "lowest",   # Does not require Admin installation
    R_flags = '/SILENT /DIR=""C:\\""'   # Install R on C Drive to avoid Program Files )
```

### Step 3: Modify ensure.R and package_manager.R
Download `ensure.R` and `package_manager.R` from this Github repo and copy/paste them into the `utils` folder that was created by `create_app()`. 

### Step 4: Modify the Inno Setup Script (if necessary)
Something funny happens to some file names that have `.` (periods) in them (e.g., `data.table`). For some reason, when `create_app` creates the Inno Setup Script (.iss) in your app folder, it may have removed any files with periods. So if you used `data.table` package (like I did), then all the associated file names with that package were changed to `datatable`. To overcome this, open the `.iss` file and find places where this might have happened and correct the names. This can be difficult to do manually, so I suggest starting with the package names and then any other files included in your app that have periods in their names. Use `Ctrl+H` to find and replace errors you might find.

### Step 5: Bundle your temporary library with your app
In your application's folder, create a new sub-folder simply called, "library". Then copy all the packages that are in your `temp_library` into this new library folder. Make sure they all copied over and that they include base R packages **and** `jsonlite`, `shiny`, and `magrittr`. 

### Step 6: Compile
Before you compile, you can optionally change the `infobefore.txt` and `infoafter.txt` files, as well as the icons used in your app. RInno provides defaults, but feel free to change them.

Once you're done modifying the Inno Setup Script and the package managing R scripts, you can compile using:
```r
compile_iss()
```
This function will attempt to download the version of R that you specified in `create_app`, so make sure you have provided your credentials in a web browser and acknowledged the internet terms of use before compiling.

### Step 7: Double-Check
The compiling process should have created a sub-folder, "RInno_installer", which is where your executable file is placed. Open your executable file and install the app on your desktop and another desktop to make sure it works as expected. 

### Troubleshooting
If you get an error after you installed and tried opening the app, there is an error log located in the `error` sub-folder of the desktop app. Read the log to determine where the error might be.
  
Some typical errors might have to do with packages missing. This could be related to the package files not being correctly compiled by Inno Setup. Check the file names to ensure that no packages had files with a `.` period in the name. You can also read the log to see if the library path is correctly using your `library` sub-folder that is within your app's directory.

