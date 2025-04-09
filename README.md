# Hands_on_projects

Subject: Request to Enable Microsoft Fabric Notebooks and Required Permissions

Hi [Admin's Name],

I hope you’re doing well.

I'm currently working with semantic models in Microsoft Fabric and need to read and interact with them using Python. Microsoft provides a built-in Python SDK called SemPy (semantic-link), which is available exclusively inside Microsoft Fabric Notebooks.

However, I’m currently unable to see or create a Notebook in my workspace. After some research and testing, it appears this might be due to one of the following reasons:

Fabric Notebooks may be disabled at the tenant level.

The workspace may not be Fabric-enabled.

My workspace role may not allow content creation (I'm possibly set as a Viewer).

To proceed, I’d appreciate your help with the following:

✅ Ensure that Microsoft Fabric is enabled for the workspace.

✅ Enable "Notebooks" under Microsoft Fabric settings in the Power BI Admin Portal.

✅ Confirm that I have at least a Member or Contributor role in the workspace so I can create and run Notebooks.

The use of Notebooks is essential here because:

I’m using Python to query semantic models directly (via fabric.read_table() and related SemPy functions).

The semantic-link package is only available inside Fabric Notebooks — it cannot be installed or used in local environments like Jupyter.

Our license is Microsoft Fabric Pro, which supports semantic model access via Notebooks in Pro-capacity workspaces.

Please let me know if you need the workspace name or any other details from my side.

Thanks so much for your support!

Best regards,
[Your Name]