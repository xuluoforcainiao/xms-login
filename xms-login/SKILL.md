---
name: xms-login
description: Automates XMS (XMS客服管理系统) SSO login via browser automation. Handles navigation to the XMS portal, SSO redirection detection, DOM-based credential injection, and login confirmation. Use when the user needs to log into XMS, authenticate with the cs-packet.i4px.com system, or perform any XMS workflow that requires a valid session.
---

# XMS 登录自动化

## Overview

Automates the SSO login flow for the XMS customer service management system. The system redirects unauthenticated users from `cs-packet.i4px.com` to an SSO login page at `sso.i4px.com`. Credentials are injected via JavaScript DOM manipulation because the login form is hidden from the accessibility tree.

## Login Workflow

### Step 1: Navigate to XMS

Open `http://cs.packet.i4px.com/` in the browser.

### Step 2: Detect SSO Redirection

Check the current page URL:
- If the URL contains `sso.i4px.com`, proceed to Step 3 (SSO login required)
- If the URL is already `cs-packet.i4px.com/index` or similar, the user is already logged in — skip to Step 4

### Step 3: Inject Credentials and Submit

Use `javascript_tool` to directly manipulate the login form DOM:

```javascript
// Fill credentials
const loginName = document.getElementById('loginName');
const loginPwd = document.getElementById('loginPwd');

if (loginName && loginPwd) {
  loginName.value = 'USERNAME';
  loginPwd.value = 'PASSWORD';

  // Trigger input events so the framework registers the values
  loginName.dispatchEvent(new Event('input', { bubbles: true }));
  loginPwd.dispatchEvent(new Event('input', { bubbles: true }));

  // Click the login button
  const loginBtn = document.querySelector('#loginBtn, .login-btn, button[type="submit"]');
  if (loginBtn) {
    loginBtn.click();
  }
}
```

> Replace `USERNAME` and `PASSWORD` with the actual credentials from configuration.

### Step 4: Confirm Login Success

Wait for the page to redirect to `cs-packet.i4px.com/index`. Verify by checking:
- Page URL contains `cs-packet.i4px.com/index`
- Page title is "XMS客服管理" or similar
- The left sidebar menu is visible (indicating successful login)

If the redirect does not occur within 30 seconds, retry Step 3 once.

## Critical Rules

- **DOM injection only**: The login form fields are invisible to the accessibility tree, so `javascript_tool` is the only reliable way to fill them
- **Trigger input events**: Simply setting `.value` is not enough — always dispatch `input` events so the frontend framework detects the change
- **Button selector fallback**: If `loginBtn` is not found by ID, try class selectors or `button[type="submit"]`
- **Already logged in**: Always check the current URL first to avoid unnecessary re-login
