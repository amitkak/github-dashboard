# GitHub Dashboard

A web-based dashboard to monitor GitHub workflows and activity for your repositories.

**Live URL:** https://amitkak.github.io/github-dashboard/

---

## Features

- **Workflow Status** - View all workflows with their last run status and next scheduled run
- **Activity Summary** - Commits, PRs, and Issues counts for 3 days, 1 week, and 1 month
- **Recent Activity Feed** - Last 10 activity items (commits, PRs, issues)
- **Multi-Repo Support** - Switch between repositories with tabs
- **Dark/Light Theme** - Toggle between dark and light modes
- **Auto-Refresh** - Data refreshes automatically every 2 hours
- **Secure Authentication** - Personal Access Token stored only in browser localStorage

---

## Supported Repositories

- `amitkak/indusglobal`
- `amitkak/igir_code`

---

## Authentication

### Getting a Personal Access Token (PAT)

1. Go to [GitHub Settings → Personal Access Tokens](https://github.com/settings/tokens)
2. Click **"Generate new token (classic)"**
3. Set an expiration date (recommended: 30-90 days)
4. Select the following scopes:
   - `repo` - Full access to repositories
   - `workflow` - Access to GitHub Actions workflows
   - `read:org` - Read organization membership
5. Click **"Generate token"**
6. Copy the token (starts with `ghp_`)
7. Paste it in the dashboard login form

### Security Notes

- Token is stored in browser's `localStorage` only
- Token never leaves your browser except to GitHub's API
- You can clear the token anytime via the Logout button or browser settings
- Token is never committed to the repository

---

## Development History

This section documents the approaches tried and decisions made during development.

### Approach 1: OAuth 2.0 with Popup Window (Failed)

**Description:** Initial implementation used GitHub OAuth with a popup window for authentication.

**Implementation:**
- User enters Client ID and Client Secret
- A popup window opens to GitHub OAuth authorization
- Code is extracted from popup URL after authorization
- Code exchanged for access token

**Problems Encountered:**
- Cross-origin restrictions prevented reading popup URL in some browsers
- GitHub OAuth popup flow is designed for localhost, not GitHub Pages
- Token exchange via fetch() was blocked by CORS
- The popup approach worked inconsistently across browsers

**Why Abandoned:** GitHub's OAuth flow is primarily designed for server-side or localhost applications. The popup-based client-side approach faced security restrictions that made it unreliable.

---

### Approach 2: OAuth 2.0 with Redirect (Failed)

**Description:** Modified the OAuth flow to use URL redirects instead of popups.

**Implementation:**
- User enters credentials → page redirects to GitHub OAuth
- GitHub redirects back with authorization code in URL
- JavaScript extracts code and exchanges for token
- Token stored in localStorage

**Problems Encountered:**
- Token exchange requires POST request to `https://github.com/login/oauth/access_token`
- GitHub blocks this request from browser due to CORS policy
- Even with proper headers, GitHub returns errors
- No server-side component available to proxy the request

**Why Abandoned:** GitHub's OAuth token exchange endpoint doesn't support CORS-enabled requests from browsers. This is a fundamental limitation of using OAuth with a purely client-side application.

---

### Approach 3: Personal Access Token (Success)

**Description:** Switched to GitHub Personal Access Token (PAT) authentication.

**Implementation:**
- User generates a PAT from GitHub Settings
- PAT is entered directly in the dashboard
- All API requests use `Authorization: Bearer {token}` header
- Token stored in localStorage for session persistence

**Advantages:**
- Simple, straightforward authentication
- No OAuth complexity or redirect issues
- Works perfectly with GitHub Pages (client-side only)
- Fine-grained permissions via scopes
- Easy to revoke and regenerate

**Disadvantages:**
- User must manually generate and manage token
- Token expires (needs renewal)
- Less "seamless" than OAuth login flow

**Why Chosen:** PAT is the most reliable authentication method for client-side applications deployed on GitHub Pages. It provides full access to GitHub's API without requiring a server-side component.

---

## Technical Details

### Architecture

- **Single HTML File** - All CSS and JavaScript embedded in `index.html`
- **GitHub REST API v3** - No GraphQL dependencies
- **No Build Step** - Pure HTML/CSS/JS, deployable directly to GitHub Pages
- **Browser Storage** - localStorage for token and theme persistence

### API Endpoints Used

| Data | Endpoint |
|------|----------|
| User Info | `GET /user` |
| Workflows | `GET /repos/{owner}/{repo}/actions/workflows` |
| Workflow Runs | `GET /repos/{owner}/{repo}/actions/workflows/{id}/runs` |
| Commits | `GET /repos/{owner}/{repo}/commits?since={date}` |
| Pull Requests | `GET /repos/{owner}/{repo}/pulls?state=all&sort=updated` |
| Issues | `GET /repos/{owner}/{repo}/issues?state=all&since={date}` |

### File Structure

```
github-dashboard/
└── index.html    # Complete dashboard (HTML + CSS + JS)
```

### Deployment

The dashboard is deployed as a GitHub Pages site.

**Deployment Steps:**
1. Create repository on GitHub
2. Push `index.html` to main branch
3. Enable GitHub Pages: Settings → Pages → Source: main branch
4. Dashboard available at `https://{username}.github.io/{repo-name}/`

---

## Future Improvements

Potential enhancements for future versions:

- [ ] Add more repositories dynamically
- [ ] Display workflow run history graphs
- [ ] Add notifications for workflow failures
- [ ] Support for GitHub GraphQL API for better performance
- [ ] Export activity data to CSV
- [ ] Add date range picker for activity queries
- [ ] Mobile-responsive improvements
- [ ] Add organization-level activity view

---

## Troubleshooting

### "Invalid token" Error

- Verify the token has not expired
- Ensure the token has the required scopes (`repo`, `workflow`, `read:org`)
- Try generating a new token

### Workflows Not Showing

- Check if the repository has GitHub Actions enabled
- Verify the token has `workflow` scope
- Ensure the repository is private and token has access

### CORS Errors in Console

- This dashboard makes direct API calls from the browser
- If you see CORS errors, the token may be invalid or expired
- Refresh the page and re-authenticate

---

## License

MIT License - Feel free to use and modify for your own projects.

---

## Credits

Created with OpenCode AI Assistant.

Built for personal use to monitor GitHub Actions workflows and repository activity across multiple repositories.
