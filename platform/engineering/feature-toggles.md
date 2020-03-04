# Feature flags (aka feature toggles, feature flippers)
VSP provides feature toggles that can be used in both vets-api and vets-website. Feature toggles enable VFS teams to test out new functionality (applications, features, VA.gov content pages, Metalsmith) in the VSP development, staging, or production environments for a set of users. Teams can enable or disable a feature for all users, a percentage of all users, a percentage of all logged in users, a list of users, or users defined in a method.

Currently feature toggles are managed by designated administrators (ask in #VSP-Platform-Support slack), but users will be allowed more access in the future. For now, tag the following groups in Slack for more information:
- Troubleshooting: #vsp-product-support
- Onboarding: #vsp-product-support
- Maintenance: #vsp-tools-fe
- Training documents: #vsp-tools-fe
- Product feedback or new feature requests: #vsp-tools-fe

Feature toggles are powered by an open-source gem called [Flipper gem](https://github.com/jnunemaker/flipper). Feature toggles:
- Allow for production toggle switching without redeploying `vets-website`
- Provide a UI for managing feature toggle behavior
- Provide code helpers for handling common UX scenarios

## Client behavior

1. During the build process, feature toggle values are retrieved and included in each of the static HTML pages.
2. When the page loads, the feature toggle client retrieves the bootstrapped values from the static HTML and renders the page using those values.
3. After the page is rendered, the feature toggle client retrieves the latest toggle values from the feature toggle service.
4. The page is updated using the latest feature toggle values.

## User experience (UX) considerations

There are a couple of ways to use release toggles, both of which have UX trade offs:

- **Use the bootstrapped values on the initial render.** The application uses the values from the static page first and _updates_ the markup if the values retrieved from the service are different. This works well for content that is not visible on the initial render of the page. Ideally, the update is either not visible to the user or comes into view using a simple CSS transition.
- **Ignore the bootstrapped values and show a loading state for the feature.** The applicaton shows a loading state for the new feature while the toggle values are retrieved from the service. This works well for content that is visible on the initial render of the page. There's no standardized approach — the way this is updated is depdendent on your UX goals.

## Enabling and disabling features

To enable or disables features, sign into the Flipper user interface (UI) at the following URLs:

|Environment|URL|
|---|---|
|Dev|http://localhost:3000/flipper/features|
|Staging|https://staging-api.va.gov/flipper/features| 
|Production|https://api.va.gov/flipper/features|

To access the Flipper UI, you must sign in using an identity-verified id.me user that is listed in [settings.yml](https://github.com/department-of-veterans-affairs/vets-api/blob/master/config/settings.yml):

```
flipper:
  admin_user_emails:
    - email@email.us
    - email1@email.us
```

<b>Notes:</b> 
- If you are not on the list, you can add yourself or your teammates. 
- If you're not sure if your account is identity-verified, you can check by going to [this page](https://www.va.gov/profile/). If you need to verify your account you'll see a "Verify with ID.me" button.

<img width="1287" alt="Screen Shot" src="https://user-images.githubusercontent.com/19188/74881655-b4d11a80-533b-11ea-8e97-fdea24c10830.png">

Once you have logged into the Flipper UI, you can perform the following actions:
- Select "Enable for everyone” or “Disable for everyone" to enable or disable the feature for all users. 
- For a gradual rollout or an a/b test you can use "Percentage of Logged in Users." "Percentage of Logged in Users" will enable the feature for the same users each time they return to the site as long as the percentage is not changed. 
- User "Percentage of Time" to enable a feature for all users for a percentage of time.
- Register a "Group" of users to enable a feature for.
- You can also roll out a feature for a select few users by adding their email address to the “Users” section. For performance reasons, the list of users is intended to be small — do not use this option for hundreds of users.

The values of each toggle are cached in memory for one minute, so it may take that long to see the effect of enabling or disabling the toggle.

## Adding a new feature toggle

Follow these steps to add and use a new feature toggle to use in `vets-website`:

1. Determine your feature toggle name.

<b>Note:</b> There are no naming conventions yet. Current examples put the application name first, such as _facilityLocatorShowCommunityCares_ and _profileShowDirectDeposit_.

2. Add the feature toggle name to `vets-website` using camel case: [/src/platform/utilities/feature-toggles/feature-toggle-query-list.json](https://github.com/department-of-veterans-affairs/vets-website/blob/master/src/platform/utilities/feature-toggles/feature-toggle-query-list.json)

 ```json
{
    "featureToggleQueryList": ["appNameThenYourFeatureName"]
}
 ```

3. Add the feature name to `vets-api` using snake case: [config/features.yml](https://github.com/department-of-veterans-affairs/vets-api/blob/master/config/features.yml)

  ```yml
 features:
   app_name_then_your_feature_name:
     description: >
       On https://www.va.gov/find-locations/ enables search box.
       This toggle is owned by the search team.
  ```

4. Submit a pull request (PR) for each feature. Crosslinking the PRs in a comment will make it easier for the reviewers to check.

5. Run `vets-api` locally. This can be done on master after your PR is merged or off of your feature branch.

6. Navigate to [http://localhost:3000/flipper/features](http://localhost:3000/flipper/features) and verify that you see your new feature name. If not, restart your rails server.

7. Use the selector helper to build a selector for your feature toggle. For example:

 ```js
 // import the toggleValues helper
 import { toggleValues } from 'platform/site-wide/feature-toggles/selectors';
 
 // use the toggleValues helper to select the toggle values list
 export const appNameThenYourFeatureName = state =>
  toggleValues(state).appNameThenYourFeatureName;
 ```

The `toggleValues` object is simply a flat list of `toggleName` and boolean key value pairs.

8. Use the feature toggle value to gate your new behavior. For example, you can use the select above in `mapStateToProps` to pass the toggle as a prop into your component.

 ```js
 function mapStateToProps(state) {
   return {
     showYourFeatureName:
       appNameThenYourFeatureName(state),
   };
 }

 ...
 // inside your connected component
 
 render() {
    const { showYourFeatureName } = this.props;
 
    return (
      { showYourFeatureName && <NewFeature /> }
    );
 }
 ```
 
Currently the feature toggles values are only available on the global redux state.

9. Use the Flipper UI to test out the toggle locally.

  ![](../../../images/platform/feature-flags/change-feature-toggle-value.png)

Updating the feature toggle state on the website requires refreshing the page. This value can take up to a minute to update in staging and production.

## Back end implementation
To check if a feature is enabled within the context of a specific user, call `Flipper.enabled?('facility_locator_show_community_cares', @current_user))`.  The user parameter is optional.

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

## Other considerations

- Each environment has its own set of feature toggle values.
- Test your feature toggle in staging before using it in production.
- Remove release toggles as soon as they are not needed.
- Make toggles that are easy to delete by gating a behavior in as few places as possible. It's often better to have blocks of repeating code that can be quickly deleted later than it is to gate several small pieces of code
