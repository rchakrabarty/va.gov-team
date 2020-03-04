# Feature flags (aka feature toggles, feature flippers)
VSP provides feature toggles that can be used in both vets-api and vets-website. Feature toggles enable VFS teams to test out new functionality (applications, features, VA.gov content pages, Metalsmith) in the VSP development, staging, or production environments for a set of users. Teams can enable or disable a feature for all users, a percentage of all users, a percentage of all logged in users, a list of users, or users defined in a method.

Currently feature toggles are managed by designated administrators (ask in #VSP-Platform-Support slack), but users will be allowed more access in the future. For now, tag the following groups in Slack for more information:
- Troubleshooting: #vsp-product-support
- Onboarding: #vsp-product-support
- Maintenance: #vsp-tools-fe
- Training documents: #vsp-tools-fe
- Product feedback or new feature requests: #vsp-tools-fe

The feature toggles are powered by an open-source gem called [Flipper gem](https://github.com/jnunemaker/flipper).

## Managing feature toggles
To enable or disables features, log into the Flipper user interface (UI) at the following URLs:

|Environment|URL|
|---|---|
|Dev|http://localhost:3000/flipper/features|
|Staging|https://staging-api.va.gov/flipper/features| 
|Production|https://api.va.gov/flipper/features|

To access the Flipper UI, you must log in using an identity-verified id.me user that is listed in [settings.yml](https://github.com/department-of-veterans-affairs/vets-api/blob/master/config/settings.yml):

```
flipper:
  admin_user_emails:
    - email@email.us
    - email1@email.us
```

<b>Notes:</b> 
- If you are not on the list, you can add yourself or your teammates. 
- If you're not sure if your account is identity-verified, you can check by going to [this page](https://www.va.gov/profile/). If you need to verify your account you'll see a "Verify with ID.me" button.

Once you have logged into the Flipper UI, you can perform the following actions:
- Select "Enable for everyone” or “Disable for everyone" to enable or disable the feature for all users. 
- For a gradual rollout or an a/b test you can use "Percentage of Logged in Users." "Percentage of Logged in Users" will enable the feature for the same users each time they return to the site as long as the percentage is not changed. 
- User "Percentage of Time" to enable a feature for all users for a percentage of time.
- Register a "Group" of users to enable a feature for.
- You can also roll out a feature for a select few users by adding their email address to the “Users” section. For performance reasons, the list of users is intended to be small — do not use this option for hundreds of users.

The values of each toggle are cached in memory for 1 minute, so it may take that long to see the effect of enabling or disabling the toggle.

<img width="1287" alt="Screen Shot" src="https://user-images.githubusercontent.com/19188/74881655-b4d11a80-533b-11ea-8e97-fdea24c10830.png">

## Front end implementation
The front end queries the `/v0/feature_toggles` endpoint ([swagger](https://department-of-veterans-affairs.github.io/va-digital-services-platform-docs/api-reference/#/site/getFeatureToggless)), which returns true/false for each feature toggle. See [Front end feature flags](https://department-of-veterans-affairs.github.io/veteran-facing-services-tools/platform/tools/feature-flags/) ("Release toggles") for more information.

## Back end implementation
To check if a feature is enabled within the context of a specific user, call  `Flipper.enabled?('facility_locator_show_community_cares', @current_user))`.  The user parameter is optional.

To initialize the feature flag in each environment add the feature name in [config/features.yml](https://github.com/department-of-veterans-affairs/vets-api/blob/master/config/features.yml):

```
---
# Add a new feature toggle here to ensure that it is initialized in all environments.
# Features are defaulted to enabled in development and test environments and disabled in all others.
# The description should contain any relevant information for an admin who may toggle the feature.

features:
  facility_locator_show_community_cares:
    description: >
      On https://www.va.gov/find-locations/ enable veterans to search for Community care by showing that option in the "Search for" box.
```
