# POC for the Image Zooming

## Prerequisites

- Winsows WSL (or Linux VM) - https://docs.microsoft.com/en-us/windows/wsl/install-win10
- Windows Terminal -  https://www.microsoft.com/en-us/p/windows-terminal/9n0dx20hk701
- Azure Storage Explorer - https://azure.microsoft.com/en-ca/features/storage-explorer/ 

## DEEP Zoom setup

- Create storage account and enable static web site
![docs](/docs/staticws.jpg)

### Split TIFF to DeepZoom Format

- Install vips tool in Linux or WSL
```
sudo apt install libvips
sudo apt install libvips-tools
```

- Run `vips dzsave` to prepare image 

```
vips dzsave HR11-1075-S11-31517-1D.tiff slides
```

It will generate `*.dzi` and multiple layers tile images

- Load all resulting tiles into storage account in `$web` container

![docs](/docs/dzi-storage.jpg)

### Prepare Web Page with OpenSeadragon

Follow [OSD GetStarted](https://openseadragon.github.io/docs/) and create index.html pointing `tileSources` to slides in DZI format and `prefixUrl` to the images coming with openseadragon library:

```
 <div class="demoarea">
        <div class="demoheading">
            OpenSeadragon Viewer With Default Settings
            <span id='consolelog'></span>
        </div>
        <div id="contentDiv" class="openseadragon"></div>
     </div>
    
    <script type="text/javascript">
        // Example
        OpenSeadragon({
            id:              "contentDiv",
            prefixUrl:       "/openseadragon-2.4.2/images/",
            tileSources:     "/slides.dzi",
            visibilityRatio: 1.0,
            constrainDuringPan: true
        });
    </script>
```

- Navigate to primary endpoint for static website


## IIIF setup
- Replace url in `docker/cantaloupe.properties` file to point to **Azure Storage Static Web Site Primary endpoint** (shown above) where TIFF file stored

```
HttpSource.BasicLookupStrategy.url_prefix = https://url>.web.core.windows.net/tiff/
```


-  create Azure Container Registry and enable admin user 
![docs](/docs/acr.jpg)


- Build docker image for [Cantaloupe IIIF server](https://cantaloupe-project.github.io/)

```
cd docker
az acr build --registry <name of ACR> --image cantaloupe:5.0.2 .
```

- Create Azure Web App and deploy Cantaloupe Container
![docs](/docs/webappcreate.jpg)

![docs](/docs/webapp.jpg)


- Set webapp configuration `WEBSITES_PORT` to port exposed by docker image
![docs](/docs/webappport.jpg)

- Start Web app and havigate to its URL
![docs](/docs/webappurl.jpg)

- To validate IIIF server navigate to image URL
```
https://<url>.azurewebsites.net/iiif/3/HR11-1075-S11-31517-1D.tiff/info.json
```

- Create `indexiiif.html` and upload it to the Storage Account, where `tileSources` should point to IIIF Server url hosting the image:

```
<div class="demoarea">
        <div class="demoheading">
            OpenSeadragon Viewer With Default Settings
            <span id='consolelog'></span>
        </div>
        <div id="contentDiv" class="openseadragon"></div>
     </div>
    
    <script type="text/javascript">
        // Example
        OpenSeadragon({
            id:              "contentDiv",
            prefixUrl:       "/openseadragon-2.4.2/images/",
            visibilityRatio: 1.0,
            constrainDuringPan: true,
            sequenceMode:       true,
            tileSources: ["https://<url>.azurewebsites.net/iiif/3/HR11-1075-S11-31517-1D.tiff/info.json"],
            "tiles": [{
                "scaleFactors": [ 1, 2, 4, 8, 16, 32, 64 ],
                "width": 1024
                }]
        });
    </script>
```

 
