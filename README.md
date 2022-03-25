# serverless-stack-policy-bug

Deploying this from new produces the following error:

```
npm run sls -- deploy --stage mtsetest

> serverless-stack-policy-bug@0.0.1 sls <redacted>\serverless-stack-policy-bug
> sls "deploy" "--stage" "mtsetest"


Deploying some-service to stage mtsetest (us-east-1)

× Stack some-service-mtsetest failed to deploy (67s)
Environment: win32, node 14.19.0, framework 3.10.0 (local), plugin 6.2.0, SDK 4.3.2
Credentials: Local, "<redacted" profile
Docs:        docs.serverless.com
Support:     forum.serverless.com
Bugs:        github.com/serverless/serverless/issues
```

If the stack already existed, updates are likely to work as expected.

Confirmed the following change fixes things:

```diff
diff --git a/lib/plugins/aws/lib/update-stack.js b/lib/plugins/aws/lib/update-stack.js
index 0bd107085..9e8385ec9 100644
--- a/lib/plugins/aws/lib/update-stack.js
+++ b/lib/plugins/aws/lib/update-stack.js
@@ -91,6 +91,11 @@ module.exports = {
       return false;
     }

+    log.info('Executing created change set');
+    await this.provider.request('CloudFormation', 'executeChangeSet', executeChangeSetParams);
+
+    await this.monitorStack('update', changeSetDescription);
+
     // Policy must have at least one statement, otherwise no updates would be possible at all
     if (
       this.serverless.service.provider.stackPolicy &&
@@ -106,11 +111,6 @@ module.exports = {
       });
     }

-    log.info('Executing created change set');
-    await this.provider.request('CloudFormation', 'executeChangeSet', executeChangeSetParams);
-
-    await this.monitorStack('update', changeSetDescription);
-
     return true;
   },
```
