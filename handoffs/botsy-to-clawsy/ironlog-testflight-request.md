# IronLog TestFlight Build Request

**From:** Botsy
**To:** Clawsy
**Date:** 2026-02-08
**Priority:** High

## Request

User wants the IronLog training app sent to TestFlight, same channel as the golf app.

You just successfully completed the TestFlight setup for the Base 90 Golf app. Now apply the same process to the training app.

## App Details

- **Name:** IronLog
- **Project Path:** `/Users/bustermejborn/Desktop/Apps/training-app/`
- **App Version:** 1.1.0
- **Bundle ID:** com.anonymous.iron-log
- **EAS Project ID:** 175a7e75-c763-49ad-b338-198837d61d0c
- **Owner:** bmejborn

## Current State

- ✅ EAS configured (eas.json exists)
- ✅ Project registered with EAS
- ✅ Git clean (no uncommitted changes)
- ✅ App is production-ready

## What to Do

Apply the same TestFlight workflow you used for the golf app:

1. **Build for iOS production:**
   ```bash
   cd /Users/bustermejborn/Desktop/Apps/training-app
   eas build --platform ios --profile production
   ```

2. **Wait for build to complete** (check status if needed)

3. **Submit to TestFlight:**
   ```bash
   eas submit --platform ios --latest
   ```

4. **Verify submission** succeeded

5. **Report back** with:
   - Build ID
   - Submission status
   - TestFlight link (if available)
   - Any issues encountered

## Notes

- Use the same Apple credentials/App Store Connect setup
- The bundle ID is different: `com.anonymous.iron-log`
- If this is the first build for this app, it may need App Store Connect setup
- Production build profile is already configured in eas.json

## Expected Timeline

Based on the golf app build:
- Build: ~15-20 minutes
- Submit: ~5 minutes
- TestFlight processing: varies (Apple's side)

## Success Criteria

- ✅ iOS build completes successfully
- ✅ App submitted to TestFlight
- ✅ User can install via TestFlight

---

Let me know if you need any clarification!

— Botsy
