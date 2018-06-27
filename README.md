# Desktop-Shiny-Apps-for-LAC
Using the RInno package in the LA County network can pose workflow issues. Follow these steps to deploy Shiny apps on secure locations.

# The Problem
You can create a wonderful Shiny application with R that processes data and empowers others to be productive, but how do you deploy it to non-R-users? Your options are either a web-based application (hosted on a Shiny server) or a desktop version. This brief tutorial shows you how to use the RInno package to turn your Shiny app into an executable file that can be installed on any Windows PC. 

More info on the RInno package can be found here: https://github.com/ficonsulting/RInno and here: https://www.ficonsulting.com/filabs/RInno

### RInno in the LA County Environment
By default, RInno creates two files that deal with making sure that your Shiny app has the packages it needs to work: `ensure.R` and `package_manager.R`. These files essentially check to see what packages are needed and uses an internet connection to install them, if not already installed. Unfortunately, we cannot expect this install process to work properly every single time because there are security protocols in place that require LA County network users to sign-in with credentials and even then the packages may fail to download or improperly install.  

# The Solution
By modifying the two files that check for and install packages, we can bypass this issue with internet connectivity/security. Also, we can bundle the packages required for the application in the executable file so that there is no need to download them from anywhere else. Make sure to use R version 3.4.3 and the latest version of RInno to create your application __files__. You may use any version to actually create your __app__, but RInno seems to work best with the latest version of R. 

### Step 1: Manage your library as you build your app
As you build your application, make note of the packages and package version you are using. Also make note of the R version you are using. Ensure that you have identified all the dependencies that your packages may need. This can be difficult when you use many packages because one package may depend on a dozen other packages, so I suggest installing the packages you are using into a separate folder. You can accomplish this by doing something like:
  
`install.packages( "ggplot2", lib = "C:/Users/e12345/Desktop/temp_library" )`

Or if you want an older version of a package, you can find its URL and download it like this:
  
`packageurl <- "http://cran.r-project.org/src/contrib/Archive/ggplot2/ggplot2_0.9.1.tar.gz"
install.packages(packageurl, repos=NULL, type="source", lib = "C:/Users/e12345/Desktop/temp_library")`
  
Assuming you have already created a folder on your desktop called, "temp_library", the above code will install the specified package and its dependencies. **Note: This is not intended to be your working library, just a separate place to manage your packages** 

You should install your app's packages in this temporary folder in addition to the three packages that RInno itself requires for it to work: jsonlite, shiny, and magrittr. You also want to make sure that all the base R packages are included in your temporary folder. This would depend on which version of R you are using.

#### Bundle your temporary library with your app
**Last tip for managing your packages:** create a new folder in your app directory called "library". You need to copy/paste all the packages your app needs into this folder before you call the `create_app` function. It could be argued that you don't really need to do the previous step of downloading packages into a "temp library" folder, but I found that this was a cleaner and more manageable workflow for me. 

### Step 2: Create your app
  
At a high level, creating your app with RInno involves:
* Installing the latest `RInno` from CRAN or Github. 
* Installing Inno Setup (to compile)
* Using `create_app()` to create the application files
* Modifying `ensure.R` and `package_manager.R` using this repo
* Compiling the app

First, run this code to create the files necessary to build an executable file:

```r
# If you don't have development tools, install them
install.packages("devtools"); require(devtools)
# Use install_github to get RInno
devtools::install_github("ficonsulting/RInno",  build_vignettes = TRUE)
require(RInno)
# Use RInno to get Inno Setup (only need to install Inno once)
RInno::install_inno()

create_app(
    app_name    = "My Cool App Name", # this just gives your app a name to use on various parts of your app  
    app_dir     = "C:/path/to/MyApp", # location of my app
    # pkgs        = c("shiny"),  # Don't use this argument
    # remotes     = c("daattali/shinyjs"), # Don't use this argument
    user_browser = "ie",  # LA County uses Internet Explorer as the standard browser.
    include_R   = TRUE,     # Download R and install it with your app
    R_version   = "3.4.3",  # Specify the version of R you want
    privilege   = "lowest",   # Does not require Admin installation
    app_icon = "la_county.ico",
    default_dir = "userdesktop",  # Install to desktop to avoid issues with servers
    R_flags = '/SILENT'   # Install R 
    )
```
The above code should have created a `utils` folder and several files in your app folder. Calling `create_app` would normally create a library folder in your app directory, but __since you already have one with all your packages in it__ it does not need to create it or override it. Check this folder to make sure that all your packages are still there.

There are other arguments and options and I suggest you use `?RInno::create_app` to read more about them. For example, you can include an installation of Google Chrome and specify that your app use Chrome as the default browser. However, doing so might drastically increase the size of your application. 

### Step 3: Modify ensure.R and package_manager.R
Download `ensure.R` and `package_manager.R` from this Github repo and copy/paste them into the `utils` folder that was created by `create_app()`. Remember, these scripts normally use an internet connection to download required packages, but because you already provided them in your own `library` folder, it does not need to go through this check. The modified versions of these scripts basically omit the `install.packages()` calls to prevent connectivity/authorization issues.  

### Step 4: Compile
Before you compile, you can optionally change the `infobefore.txt` and `infoafter.txt` files, as well as the icons used in your app. RInno provides defaults, but feel free to change them.

Once you're done modifying the Inno Setup Script and the package managing R scripts, you can compile using:
```r
compile_iss()
```
This function will attempt to download the version of R that you specified in `create_app`, so make sure you have provided your credentials in a web browser and acknowledged the internet terms of use before compiling.

### Step 5: Double-Check
The compiling process should have created a sub-folder, "RInno_installer", which is where your executable file is placed. Open your executable file and install the app on your desktop and another desktop to make sure it works as expected. 

### Troubleshooting
If you get an error after you installed and tried opening the app, there is an error log located in the `error` sub-folder of the desktop app. Read the log to determine where the error might be.
  
Some typical errors might have to do with packages missing. This could be related to the package files not being correctly compiled by Inno Setup. You can also read the log to see if the library path is correctly using your `library` sub-folder that is within your app's directory.

