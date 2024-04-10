# Dialog tags

Dialogs are _tagged_ using classifiers. {{ speechsense-name }} determines whether the dialog has certain key phrases, classifies the dialog, and adds a tag to it. For example, there are tags that show the agent has thanked the customer for waiting or that the customer has contacted support a second time.

In the transcript, you can view which tags were assigned to the agent's and customer's channels. If you click a tag, the words it refers to will be highlighted with the appropriate color.

## How to assign tags {#attach}

After a tag is created, [activate it in the project](../operations/project/tag/change.md#activate-tag) so it can be assigned to a recording. {{ speechsense-name }} searches for key phrases and assigns a tag only after its activation. If activation is disabled, tags will remain in the dialogs they are already assigned to and will not be added to new ones.

After a tag is activated, [assign a channel to it](../operations/project/tag/change.md#tag-channel). Depending on the channel, {{ speechsense-name }} will search for keywords in different parts of the recording:

* **Agent and customer**: Anywhere in the recording.
* **Agent**: Agent's speech only.
* **Customer**: Customer's speech only.

For example, if the tag has the **Customer** channel assigned, and the tag keyword is present in the agent's speech only, the tag is not given to the call recording.

No matter what tags, a channel must be configured at the [project](resources-hierarchy.md#project) level. This allows for flexible tag assignment management across different projects.

## Tag groups in {{ speechsense-name }} {#sets}

{{ speechsense-name }} has three tag groups:

* _System tags_: Preset tags. Each project has the same system tags. For such tags, you can only change the channel and the activation status. You cannot create, edit, or delete system tags.

* _Space tags_: Tags the user creates within a [space](resources-hierarchy.md#space). Such tags are available in all projects within the space. After a space tag is created, you activate it and assign a channel to it at the project level.

* _Project tags_: Tags the user creates and configures at the project level. Different projects can have different tags.

To create, edit, or delete tags, follow [this guide](../operations/index.md).

## Where tags are displayed {#display}

* On the **Tags** tab in the project or space.
* In the [dialog filters](dialogs.md#filters) available on the **Dialogs** tab and in the list of [estimation parameters in reports](reports.md#parameters).
* On the page of the dialog the tag is assigned to.
