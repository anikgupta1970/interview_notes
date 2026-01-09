Situation/Challenge:
We had a functionality that needed to be available only for the mobile app and not for the web. The challenge was that we could not modify the existing API endpoint, so a direct change wasn’t possible.

Solution:
I introduced API versioning. By creating a new version of the API specifically for the mobile app, I was able to implement the required functionality without affecting the web users. The mobile app would call the new version, while the web continued to use the existing one.

Result/Impact:
This approach allowed us to maintain backward compatibility, ensure web stability, and deliver mobile-specific features without disrupting existing users.
_____________________________________

Use Case: Managing long-running tasks and token expiry in a Spring Boot application.

Situation/Challenge:
We had a scenario where some backend tasks, like data exports, could run longer than the JWT token TTL (e.g., 6-minute export with a 5-minute token expiry). Without handling this, the task could fail if the backend enforced token expiration strictly, or we would have to compromise security by extending token TTL globally.

Solution:
I leveraged Keycloak’s token refresh mechanism:

The frontend obtained both an access token and a refresh token from Keycloak.

For long-running tasks, the backend could request a new access token using the refresh token whenever the original JWT expired mid-task.

This ensured that tasks could complete without requiring users to re-login, while maintaining secure, time-bound token usage.

Result/Impact:

Long-running operations completed successfully without breaking security.

No need to extend JWT TTL globally, preserving the principle of least privilege.

Maintained seamless user experience across mobile and web apps.

_______________________________________

Use Case: Dynamic Feature Toggle for Clients

Problem / Challenge:
We needed a way to enable or disable certain features dynamically for different clients (mobile vs web, or even specific user groups) without deploying a new version of the application or modifying the API endpoints. For example:

A new UI element should be visible only to mobile users.

A beta feature should be available only for certain users.

Solution:

Introduce a Feature Flag System:

Created a feature_flags table in the database:

feature_name	client_type	enabled	rollout_percentage	notes
new_ui	MOBILE	true	100	mobile beta UI
quick_export	WEB	false	0	not released

client_type can be MOBILE, WEB, or ALL.

rollout_percentage allows gradual feature rollout for A/B testing.

Service Layer Logic in Spring Boot:

Expose a FeatureToggleService that checks whether a feature is enabled for a given client or user.

Example:

public boolean isFeatureEnabled(String featureName, String clientType, String userId) {
    FeatureFlag flag = featureFlagRepository.findByFeatureNameAndClientType(featureName, clientType);
    if (flag == null || !flag.isEnabled()) return false;
    
    // Optional: handle gradual rollout
    if (flag.getRolloutPercentage() < 100) {
        int hash = Math.abs(userId.hashCode() % 100);
        return hash < flag.getRolloutPercentage();
    }
    return true;
}


Controller / API Usage:

In your API or service layer, check the feature toggle before returning certain fields or allowing certain actions:

if (featureToggleService.isFeatureEnabled("new_ui", clientType, userId)) {
    response.setNewUiEnabled(true);
}


Dynamic Updates Without Deploy:

Admins can enable/disable features by updating the database or via a central config service.

No code changes or API redeploys are required.
