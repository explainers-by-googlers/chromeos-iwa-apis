# [Self-Review Questionnaire: Security and Privacy](https://w3c.github.io/security-questionnaire/)

Security and privacy questionnaire for the
[Window Shape API](https://github.com/explainers-by-googlers/chromeos-iwa-apis/blob/main/window-shape-explainer.md).

--------------------------------------------------------------------------------

1.  **What information does this feature expose, and for what purposes?**

    N/A. This API does not expose any information to the web origin. It is a
    write-only API that allows allowlisted Isolated Web Apps to request a custom
    window shape from the OS window manager.

2.  **Do features in your specification expose the minimum amount of information
    necessary to implement the intended functionality?**

    Yes. The API is write-only and does not allow querying the current window
    shape or any OS-level display details. It exposes no information.

3.  **Do the features in your specification expose personal information,
    personally-identifiable information (PII), or information derived from
    either?**

    N/A. This feature does not expose any information, including personal or
    PII.

4.  **How do the features in your specification deal with sensitive
    information?**

    N/A. This feature does not handle or expose any sensitive information.

5.  **Does data exposed by your specification carry related but distinct
    information that may not be obvious to users?**

    N/A. No data is exposed by this API.

6.  **Do the features in your specification introduce state that persists across
    browsing sessions?**

    No. The window shape is set dynamically at runtime and is ephemeral. When
    the window is closed, the shape is discarded. Reopening the app starts it
    with the default rectangular window.

7.  **Do the features in your specification expose information about the
    underlying platform to origins?**

    No. It does not expose information about the platform's window manager,
    screen configuration, or graphics stack.

8.  **Does this specification allow an origin to send data to the underlying
    platform?**

    Yes. It allows the origin to send a list of rectangles (`DOMRectReadOnly`)
    representing the requested window shape to the host operating system's
    window manager.

9.  **Do features in this specification enable access to device sensors?**

    No.

10. **Do features in this specification enable new script execution/loading
    mechanisms?**

    No.

11. **Do features in this specification allow an origin to access other
    devices?**

    No.

12. **Do features in this specification allow an origin some measure of control
    over a user agent's native UI?**

    Yes. It allows the application to control the shape of its native window,
    effectively masking parts of the window and creating non-rectangular
    boundaries. This is restricted to windows in the
    [`"unframed"`](https://github.com/WICG/manifest-incubations/blob/gh-pages/unframed-explainer.md)
    display mode (where native title bar and borders are already removed).

13. **What temporary identifiers do the features in this specification create or
    expose to the web?**

    N/A. No temporary identifiers are created or exposed.

14. **How does this specification distinguish between behavior in first-party
    and third-party contexts?**

    The API is only exposed to top-level contexts that are allowlisted Isolated
    Web Apps (IWAs). It is not available to third-party or cross-origin iframe
    contexts.

15. **How do the features in this specification work in the context of a
    browser’s Private Browsing or Incognito mode?**

    N/A. IWAs do not support Incognito mode. If they did, the behavior would be
    identical as the API is entirely ephemeral and session-based with no
    persistent storage.

16. **Does this specification have both "Security Considerations" and "Privacy
    Considerations" sections?**

    Yes, here is
    [the relevant section in the explainer](https://github.com/explainers-by-googlers/chromeos-iwa-apis/blob/main/window-shape-explainer.md#security--privacy-considerations).

17. **Do features in your specification enable origins to downgrade default
    security protections?**

    N/A.

18. **What happens when a document that uses your feature is kept alive in
    BFCache (instead of getting destroyed) after navigation, and potentially
    gets reused on future navigations back to the document?**

    The feature applies to the window, not to the document. Navigation within
    the window will not affect the current window shape.

    It's worth noting that out-of-scope navigation is not allowed within an IWA
    window. The browser will open a new tab instead.

19. **What happens when a document that uses your feature gets disconnected?**

    The Mojo pipe to the browser process is closed, and any pending promises are
    rejected. The window shape itself is a property of the OS window and will
    persist for the remaining lifetime of the OS window, or until the new active
    document modifies it.

20. **Does your spec define when and how new kinds of errors should be raised?**

    Yes.

21. **Does your feature allow sites to learn about the user's use of assistive
    technology?**

    N/A.

22. **What should this questionnaire have asked?**

    *   Are there limits to shape complexity?

    Yes, limited to 10,000 rectangles to prevent performance degradation and
    abuse.

