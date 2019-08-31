# distributed-open-source-contributions
This document describes a distributed revenue sharing model for open source contributions.

# Problem statement
Open source developers spend a generous amount of time in developing and maintaining open source
projects. **But, the currently popular monetary compensation models are skewed in favor of only a few open
source projects which are popular and being used directly by the sponsors.** Due to this, the transitive dependencies of
those open source projects are not able to attract enough monetary benefits.

*(Some of these transitive dependencies may be detectable, for example, via `package.json` for Node.js projects. While
some others may be getting used in background and hence they are not automatically detectable, for example, a tool like
GIMP.)*

# Proposed solution
Every open source project should be able to share a specific percentage of the sponsorship amount they receive, to the
other open source projects.

# Description

## Assumptions
For simplicity of discussions, let's assume that:
  * "GitHub Sponsors" has become a popular way in which people / companies sponsor the open source developers /
    projects.
  * There is no transaction fees involved.
  * All the data we handle in "Contribution sharing rules" section is in valid / complete / consistent state. The
    described revenue sharing model can be tweaked and made robust to handle any inconsistencies in the system. Hence,
    skipping those details for now.

## Initial scope
**For simplicity of discussions, let's consider that:**
  * **The proposed solution is applied only within this closed system, i.e., GitHub.**

## Terms and their meanings
  * **Project**  
    An open source project on GitHub.
  * **Project-partner**  
    A project-partner is a GitHub account representing a person or a company who is marked as a project-partner for the
    project (being discussed in context). For most of the use cases, a project-partner would be the same as a project
    team member / collaborator. The project-partner would need to have a monetary account (Bank account, PayPal account
    etc) to receive the sponsorships.
  * **Transitive project**  
    A project might be built on top of other projects. Those other projects might be built on top of many more projects.
    All of those other projects are called transitive projects.

    The list of these transitive projects might be:
    * detected automatically (for example, via `package.json` for Node.js projects)
    * some or all of those automatically detected transitive projects might be removed from the list (by not giving
      them any weightage)
    * more transitive projects might be added manually as mentioned in this proposal.
  * **Transitive project-partner**  
    A project-partner of a transitive project (being discussed in context).

## Sample workflow
* Let's consider that we have a Node.js based open-source project at:
  `https://github.com/webextensions/awesome-contributions`
* A considerate company, which makes use of the `awesome-contributions` project, decides to sponsor that project via
  "GitHub Sponsors" with a **contribution of 200 USD per week**.
* The team behind the `awesome-contributions` project has used many other open source projects, without which
  `awesome-contributions` couldn't have been built. That team wishes to share the contributions it receives with those
  open source projects.

### **Contribution sharing rules**
The GitHub platform would have the following data stored for `awesome-contributions` with following "Contribution
sharing rules". These rules would be accessible / modifiable via appropriate GUI.
```js
{
    "contributionsShare": {
        // Out of all the sponsorship amount received via "GitHub Sponsors",
        // this percentage of the amount would be shared to other projects
        // listed in "shareContributionsWeightages". The remaining sponsorship
        // amount would be transferred to the accounts listed in
        // "projectPartnersWeightages".
        // Minimum value: 0
        // Maximum value: 100
        // Default value: 50  (if "projectPartnersWeightages" exists)
        //                100 (if "projectPartnersWeightages" doesn't exist)
        "sharePercententageOfContributionsReceived": 50,
        // This means:
        //     * 50% of the sponsorship amount would go to the project-partners
        //       of "awesome-contributions" (effectively 100 USD in a week)
        //     * 50% of the sponsorship amount would go to the transitive
        //       project-partners for the transitive projects mentioned under
        //       "shareContributionsWeightages" (effectively 100 USD in a week)

        // Note that these weightages would be mapped to appropriate percentages
        // by the implementation in runtime (In this example, since the
        // weightages add up to 100, the percentages are going to be the same)
        "projectPartnersWeightages": {
            // Using "@" symbol to indicate that it is a project-partner account
            "@webextensions": 60,

            // "*" indicates remaining project-partners, who are not listed in
            // the object "projectPartnersWeightages"
            "*": 40

            // This means:
            //     * @webextensions would get 60 USD in a week
            //     * Other project-partners would get, in total, 40 USD in a week
            //       and those 40 USD would be equally distributed among them
        },

        // Note that these weightages would be mapped to appropriate percentages
        // by the implementation in runtime (In this example, since the
        // weightages add up to 100, the percentages are going to be the same)
        "shareContributionsWeightages": {
            // Notes:
            //     * Any of the projects can be included here. These may or may
            //       not be part of dependencies / devDependencies in
            //       package.json.
            "eslint/eslint": 1,
            "facebook/react": 0.95,
            "GoogleChrome/puppeteer": 80,
            "jquery/jquery": 0.05,
            "webextensions/live-css-editor": 8,

            // "*" indicates remaining projects listed in "package.json"
            // Default value: Sum of all other listed weightages
            //                (effectively, default value maps to 50% of
            //                total "shareContributionsWeightages")
            "*": 10

            // This means:
            //     * eslint/eslint would get 1 USD in a week
            //     * facebook/react would get 0.95 USD in a week
            //     * GoogleChrome/puppeteer would get 80 USD in a week
            //     * jquery/jquery would get 0.05 USD in a week
            //     * webextensions/live-css-editor would get 8 USD in a week
            //     * Other transitive projects would get, in total, 10 USD in a week
            //       and those 10 USD would be equally distributed among them
        }
    }
}
```

**Other projects would also have their own "Contribution sharing rules" which would distribute the sponsorship amount
even further.**

# Notes
* This document is currently limited to an on-paper proof-of-concept for discussions.

* By default, a project-partner is in deactivated mode for "receive sponsorship" functionality. The project-partner can
  provide their monetary account details (Bank account, PayPal account etc) to GitHub and activate the "receive
  sponsorship" functionality.

* A project would always have "receive sponsorship" functionality turned on. But, how it behaves would depend on various
  factors described here.  

  If "Contribution sharing rules" have been provided:
    * If there is at least one project-partner for that project or transitive projects for whom "receive sponsorship" is
      active, the "Contribution sharing rules" would get applied.
    * Else, the project cannot be sponsored.

  Else ("Contribution sharing rules" have not been provided)
    * If transitive projects can be automatically detected (eg: via `dependencies` / `devDependencies` in
      `package.json` for Node.js projects):
      * If there is at least one transitive project-partner with "receive sponsorship" functionality turned on:
        then 100% of the sponsorship received would be passed on to those transitive projects with an equal share
        (`shareContributionsWeightages`) to each one of the applicable projects.
      * Else, the project cannot be sponsored.
    * Else (transitive projects cannot be detected), the project cannot be sponsored.

* How the sponsorship amount should be distributed, would depend on the latest updates to the "Contribution sharing
  rules" for the project and transitive projects. So, if a project's or project-partner's name is removed from the list,
  then they would stop getting any share via the project's contributions.

# Examples of edge cases and the solutions:
* **Circular loops**  
  Suppose:
  * `project-a` contributes 50% of the sponsorship it earns to `project-b`
  * `project-b` contributes 25% of the sponsorship it earns to `project-a`

  A person sponsors 100 USD to `project-a`.
  The `project-a` passes on 50 USD to `project-b`.
  Now, as the `project-b` receives 50 USD, it sees that it is supposed to pass 25% of it to `project-a`.
  But, since such a payment to `project-a` would eventually form a circular loop, we mark the
  `shareContributionsWeightages` of `project-a` as 0 for this case and continue the flow if there are other packages.
  Otherwise, we will stop at this point.

  The same solution is applicable for circular loops involving 3 or more packages.

* **Keep the change**  
  It is possible that the sponsorships would be getting re-distributed in very small amounts. A project might be earning
  sponsorships transitively and it may receive amounts like 0.001 USD from a million sponsors, which would be equivalent
  to 1000 USD. Hence, the "least count" for this system would need to be set to a practically useful small enough value.

# Questions and Answers
**Should people need to manually set the distribution? Can this be automatically generated?**  
The `shareContributionsWeightages` may be automatically generated via some score generating tool (eg: Imagine an
internet-wide, extended version of https://github.com/sourcecred/sourcecred).

**But,** then people would try to figure out and follow some approaches to improve their score. Those approaches, if
done specifically to improve the score, may not lead to an increase in the value their project brings to the same
extent.

**So,** those values may be generated by such tools, but they must be moderated / confirmed by humans (respective
project owners), and then there wouldn't be any fixed approaches which would be able to trick the system all the time,
and hence mitigating the misuse.

**Why not distribute the sponsorship amount in terms of percentages?**  
If we wish to use percentages, rather than `shareContributionsWeightages` then either the user would need to ensure that
the values sum up to 100 or appropriate warnings would need to be added. On top of it, the status of transitive projects
and transitive project-partners may change from time to time based on various criteria described above.

So, in this model, things are kept simple for the user and these numbers can be changed easily without breaking data
integrity. Of course, if this proposal is implemented, then these values may be represented in terms of percentages via
appropriate GUI.

**Would there be a transaction fees?**  
Ideally, no. But, practically, the system, which implements this solution, may need to add some transaction fees to make
the overall solution maintainable and scalable. That transaction fees should only be charged at the first instance when
the sponsorship money enters into the system. The redistribution of that sponsorship money via above explained workflow
should not be charged at all.

# Extra thoughts
* In the assumptions mentioned in this proposal, it is assumed that this revenue sharing model depends on the success of
  "GitHub Sponsors". But, potentially, this solution can also lead to increase in popularity of "GitHub Sponsors"
  because it may lead to more people receiving monetary benefits, leading to more people talking about it, leading to
  more individuals / companies sponsoring / contributing monetarily, leading to a cultural change around open source
  development.

# Far-fetched thoughts
<details>
  <summary>Ideal solution</summary>
  <p>

  In the ideal solution, this model would be implemented by a not-for-profit organization, consisting of prominent open
  source contributors and companies which support open source development. The organization would have a platform, say
  open-source-contributions.org which would handle the contributions / sponsorships and the data mentioned above (like
  the accounts of project-partners and "Contribution sharing rules").

  GitHub, npm, GitLab, BitBucket and various other platforms would connect to open-source-contributions.org and handle
  the operations related to sponsorships and other project details.

  </p>
</details>

# Triggers which lead to this document
* https://github.com/sponsors
* https://twitter.com/feross/status/1166924812129722368

# Useful references
* [Open Collective](https://opencollective.com)
* [BackYourStack](https://backyourstack.com)
* [SourceCred](https://sourcecred.io)

# Disclaimer
* This document is not affiliated with BitBucket, GitHub, GitLab, npm and any other existent entities mentioned above.
* The examples / references to these platforms / companies are solely used for making the discussions relatable to open
  source development community.
