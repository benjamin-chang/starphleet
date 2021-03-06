### Overview

Adding the ability, within starphleet, to publish static assets in such a way as to 
optimize their delivery using a CDN. This proposes an additional configuration to the
existing service orders structure that can be used to trigger static asset deployment
after the successful build of a service.

### Required Configuration

#### Starphleet Global (HQ Level)
* `STATIC_ASSET_BUCKET` - The name of the s3 bucket (must exist) to which the static assets
  will be delivered once built. This bucket must exist for this feature to function
* `STATIC_DISTRIBUTION_ID` - The cloudfront distribution identifier used to serve the static
  assets from the aforementioned bucket.  This id will be used to issue invalidation requests
  as needed.
* `STATIC_DELIVERY_ROOT_URL` - The cloudfront url used to serve the s3 bucket indicated above,
  this serves to facilitate the creation of asset urls. It's possible that this only serves
  as documentation of the root static asset url.

#### Starphleet Orders
* `publish_static` - allows for the optional specification of a directory relative to the
  service repo root, defaulting to `<service repo>/static`


### Deployment Process
* triggered by orders containing `publish_static` 
* after normal build process for service, the directory indicated by `publish_static`
  will be deployed to s3 in two ways
      * by service name: s3://static.bucket.name/service.name/
      * by sha: s3://static.bucket.name/service.name-\<head sha\>/
* `distribution` will be sent an invalidation request for /<service name> which 
  intends to invalidate only the files provided for this service.

### Asset Access
Once the deployment process has been completed, assets will be available as follows:

      http(s)://<STATIC_DELIVERY_ROOT_URL>/<service name>/<static asset path>

and

      http(s)://<STATIC_DELIVERY_ROOT_URL>/<service name-head sha>/<static asset path>


#### Example

Having `STATIC_DELIVER_ROOT_URL=static.mydomain.com`

Given a service repository with the following structure:

      root-
      | - static-assets/
          | - img/
              | - my_asset.gif

And an starphleet orders file in an HQ directory named `my_static_app` containing:

      publish_static /static-assets

The following will be valid urls post publising the service having HEAD sha of
`9dce20983bde406ed10d9ba01b2b94312e0e82b4`:

      http(s)://static.mydomain.com/my_static_app/img/my_asset.gif

      http(s)://static.mydomain.com/my_static_app-9dce20983bde406ed10d9ba01b2b94312e0e82b4/img/my_asset.gif

#### Notes

* each deployment will result in the creation of a new 'directory' named using the
  sha of the repo HEAD at deployment, this means each distinct version of the
  service will have it's assets deployed and accessible in perpetuity.
* each deployment will update the existing 'service name' directory meaning that
  as the CDN returns to the s3 bucket to refresh expired assets it will get updated
  assets for any assets served directly by service name e.g. `http(s)://static.mydomain.com/my_static_app/img/my_asset.gif`
