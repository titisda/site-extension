# PROBLEMS OPENING THE EXTENSION
Microsoft have introduced a bug in the kudu portal that the extension relies on, until that is resolved you have to browse to the extension using: `https://[websitename].scm.azurewebsites.net/letsencrypt`

See https://github.com/sjkp/letsencrypt-siteextension/issues/134 for more info. 

# Let's Encrypt Site Extension
[![Build status](https://ci.appveyor.com/api/projects/status/mdg0e31rp0qdu16m/branch/master?svg=true
)](https://ci.appveyor.com/project/sjkp/letsencrypt-siteextension)

This Azure Web App Site Extension enables easy installation and configuration of [Let's Encrypt](https://letsencrypt.org/) issued SSL certifcates for you custom domain names. 

The site extension requires that you have configured a DNS entry for you custom domain to point to Azure Web App. 

## How to install
https://github.com/sjkp/letsencrypt-siteextension/wiki/How-to-install

## Known Issues
* This site-extension is **NOT** supported by Microsoft it is my own work based on https://github.com/ebekker/ACMESharp and https://github.com/Lone-Coder/letsencrypt-win-simple - this means don't expect 24x7 support, I use it for several of my own smaller sites, but if you are running sites that are important you should consider spending the few $ on a certificate and go with a Microsoft supported way of enabling SSL, so you have someone to blame :) 
* Note that Let's Encrypt works by providing automated certificates of a short (currently three month) duration. This extension is BETA SOFTWARE. You will need to keep this extension updated or risk losing SSL access when your certificate expires.
* There are presently no email notifications for certificate renewal problems, but Lets Encrypt will email you weeks in advance of certificate expiration, when the extension is running properly the certificate should have been renewed before Let's Encrypt send the reminder email.
* Due to rate limiting of Let's Encrypt servers, you can only request five certificates per domain name per week. Configuration errors or errors in this site extension may render you unable to retrieve a new certificate for seven days. If up-time is critical, have a plan for deploying a SSL certificate from another source in place.
* No support for multi-region web apps, so if you use traffic mananger or some other load balancer to route traffic between web apps in different regions please dont use this extension. 
* If you publish your project from Visual Studio with the "Delete Existing files" option, you will remove the web jobs the site extension uses to renew the certificate once they expire every 3 months (you can renew them manually or install the site extension again after publish). 
* The site-extension will not work with [Azure App Service Local Cache](https://azure.microsoft.com/en-us/documentation/articles/app-service-local-cache/)

## How to troubleshoot
https://github.com/sjkp/letsencrypt-siteextension/wiki/Troubleshoot

## This is Beta Software
Please take note that this Site-Extension is beta-software, so use at your own risk.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYLEFT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Semi-Automated Installation
With the semi-automated installation you manually add the site extension to your web app. Open the extension and manually click through the 3 step dialog. 

Once this process is complete your custom domain for the site is setup with a Let's Encrypt issued SSL certificate. 

## Fully-Automated Installation
To use the Fully Automated Installtion the following Web App settings must be added. 

| Key |	Value
|-----| ----
| letsencrypt:Tenant |	The tenant name e.g. myazuretenant.onmicrosoft.com
| letsencrypt:SubscriptionId |	The subscription id
| letsencrypt:ClientId	| The value of the clientid of the service principal
| letsencrypt:ClientSecret	| The secret for the service principal
| letsencrypt:ResourceGroupName |	The name of the resource group this web app belongs to
| letsencrypt:ServicePlanResourceGroupName |	The name of the resource group with the app service plan that hosts the web app, if the app service plan is in the same plan as the web app, then this property is optional. 
| letsencrypt:AcmeBaseUri |	The url to Let's Encrypt servers e.g. https://acme-v01.api.letsencrypt.org/ or https://acme-staging.api.letsencrypt.org/ (defaults to this)
| letsencrypt:Email	| The Email used for registering with Let's Encrypt
| letsencrypt:Hostnames |	Comma separated list of custom hostnames (externally hosted setup with CNames), that should automatically be configured for the site.
| letsencrypt:WebRootPath | Use this setting, if you are not serving the website from site\wwwroot, then you can sepecify the other folder that serves your website here - should be in the format d:\home\site\wwwroot\public or where ever your files are located on the web server. 
| letsencrypt:DisableWebConfigUpdate | true / false, defaults to false, set this to true if you don't want the site extension to write the default webconfig to wwwroot\.well-known\acme-challenge
| letsencrypt:SiteSlot | Use this setting if you want to use the extension to setup SSL certificate for deployment slots, the value should be the name of the slot (and the extension should be installed in that slots kudu portal)
| letsencrypt:UseIPBasedSSL | Set to true if you want to use IP Based SSL (required by some older clients). Defaults to false, which results in SNI. 
| letsencrypt:RenewXNumberOfDaysBeforeExpiration | Set to an integer defining the number of days before expiration the certificates should be renewed. Defaults to 22 days before expiration, as letencrypt sends reminder emails 20 days before

As it can be seen from the list of App Settings a service principal is needed. The service principal must be assigned permissions to the web app, that is required as the extension use it for installing and updating the certificate. (If two resource groups are used, the app service principal must have access to both). 

Besides the App Settings, the two Azure Web Job required connection strings ```AzureWebJobsStorage``` and ```AzureWebJobsDashboard``` must also exists, as the extension relies on an internal Web Job to renew the certificates once they expire. 

To see an example of an ARM template installation look at [azuredeploy.json](LetsEncrypt.ResourceGroup/Templates/azuredeploy.json)


