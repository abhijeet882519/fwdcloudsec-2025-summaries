# Not So Secret: The Hidden Risks of GitHub Actions Secrets

**Video Link**: [Watch on YouTube](https://www.youtube.com/watch?v=k3DBur7iEHM)

- **Author**: Emran Alvita (Zello)
- **Talk Type**: Security

## Summary

This presentation exposes critical security vulnerabilities in GitHub Actions secret management, demonstrating how any user with write access to a repository can exfiltrate secrets despite GitHubs masking protections. The speaker provides practical solutions using deployment environments and OIDC configurations to properly secure CI/CD credentials and prevent unauthorized access to sensitive information.

## Key Points

- Any user with write access can extract GitHub repository secrets by modifying workflows on new branches
- GitHub branch protection rules do not protect secrets from unauthorized access
- Simple encoding (base64) bypasses GitHubs automatic secret masking in workflow outputs
- Deployment environments with protection rules provide proper access control for secrets
- OIDC with properly configured trust policies offers better security than long-lived credentials
- Workflow triggers and branch restrictions are critical components of overall security strategy

## Technical Details

**GitHub Actions Security Model Flaws:**

*Repository Secrets Vulnerability:*
- Repository secrets accessible to any workflow execution within that repository
- Workflow definitions taken from the branch where execution occurs
- Users with write access can create branches with modified workflows
- Branch protection rules only apply to merging, not workflow execution

*Attack Methodology:*
1. Create new branch with modified workflow
2. Replace intended actions with secret exfiltration commands
3. Execute workflow on the new branch
4. Use encoding (base64) to bypass GitHubs secret masking
5. Extract credentials from workflow output logs

*Example Attack Workflow:*
```yaml
- name: Exfiltrate secrets
  run: |
    echo ${{ secrets.AWS_ACCESS_KEY }} | base64
    echo ${{ secrets.AWS_SECRET_KEY }} | base64
```

**Security Solutions:**

**1. Deployment Environments:**
- Alternative to repository secrets with enhanced protection capabilities
- Support configurable protection rules controlling access
- Can restrict environment usage to specific branches (e.g., main branch only)
- Provide same functionality as repository secrets but with access controls

*Environment Configuration:*
- Create production environment in repository settings
- Configure protection rule: "Only allow deployments from main branch"
- Move secrets from repository level to environment level
- Modify workflows to reference the protected environment

*Workflow Modification:*
```yaml
jobs:
  deploy:
    environment: production  # References protected environment
    runs-on: ubuntu-latest
```

**2. OIDC Configuration:**
- Replace long-lived credentials with short-lived tokens
- Requires properly configured trust policies in cloud provider
- Eliminates stored credentials in GitHub entirely
- Supports major cloud providers (AWS, Azure, GCP)

*AWS OIDC Setup:*
```yaml
- name: Configure AWS credentials
  uses: aws-actions/configure-aws-credentials@v2
  with:
    role-to-assume: arn:aws:iam::123456789012:role/GitHubActionsRole
    aws-region: us-east-1
```

**Trust Policy Security:**

*Vulnerable Configuration:*
```json
{
  "StringEquals": {
    "token.actions.githubusercontent.com:sub": "repo:org/repo:*"
  }
}
```
- Wildcard allows any branch to assume role
- Same attack vector as repository secrets

*Secure Configuration:*
```json
{
  "StringEquals": {
    "token.actions.githubusercontent.com:sub": "repo:org/repo:ref:refs/heads/main"
  }
}
```
- Restricts role assumption to main branch only

**Combined Environment + OIDC Security:**
- Use deployment environments even with OIDC configuration
- Trust policy must reference GitHub environment in subject claim
- Provides defense in depth with multiple security layers

*Enhanced Trust Policy:*
```json
{
  "StringEquals": {
    "token.actions.githubusercontent.com:sub": "repo:org/repo:environment:production"
  }
}
```

**Attack Scenarios and Mitigations:**

*Scenario 1: Repository Secrets*
- Attack: Create branch, modify workflow, extract secrets via base64 encoding
- Mitigation: Replace with deployment environments

*Scenario 2: Vulnerable OIDC*
- Attack: Modify workflow to assume role from any branch
- Mitigation: Restrict trust policy to specific branch

*Scenario 3: Environment Bypass Attempt*
- Attack: Remove environment reference but keep role assumption
- Result: Role assumption fails due to trust policy restrictions

**Workflow Triggers Security:**
- Trigger configuration always comes from default branch
- Manual triggers (workflow_dispatch) allow execution on any branch
- Review trigger types to understand attack surface
- Different trigger types have varying security implications

**Detection and Monitoring:**
- Workflow execution logs show branch information and outputs
- Past runs may be deletable depending on permissions
- Unknown retention period for workflow run history
- GitHub audit logging may provide additional visibility
- Monitor for unusual workflow execution patterns

**Open Source Considerations:**
- Pull requests from forks have different secret injection behavior
- Fork-based PRs have additional security nuances not covered
- Different security model for external contributor workflows

**Best Practices:**
- Never use repository secrets for sensitive credentials
- Always use deployment environments with protection rules
- Prefer OIDC over long-lived credentials where possible
- Combine environment protection with trust policy restrictions
- Review and restrict workflow triggers appropriately
- Implement monitoring for unusual workflow execution patterns
- Regular audit of repository access and workflow configurations

**Enterprise Implications:**
- Secrets exposure can lead to cloud account compromise
- CI/CD pipeline security critical for software supply chain
- Need for comprehensive GitHub Actions security policies
- Integration with existing security monitoring and SIEM systems

## Full Transcript

The speaker is Amaran. Um the talk is not so secret the hidden risks of GitHub action secrets. So please give it up for Amran. Hello forward cloud sec. Uh I'm super excited to be here today. My name is Emran Alvita. I'm director of security engineering at Zello. Uh we we build pushto talk voice solution for frontline workers. And I'm here today to talk about um uh security of GitHub secrets. Our kind of sample scenario here is imagine a small or small company or a startup that uses GitHub for version control and they use GitHub actions for CI/CD workflows. Those workflows obviously require credentials to do anything in their cloud environments. So a first workflow they that they might create might look something like this. This is obviously terrible because we have these hard-coded credentials in the workflow. So, anybody who looks at this will say, "Oh, what's happening? Let's get rid of them." And maybe you'll you might decide to use uh GitHub repository secrets to replace this hard-coded credentials with just references. It's fairly easy to do. uh you can go into the settings on on the repository um go into secrets and variables and then configure the secrets um there and then your workflow changes slightly but not by much. So our second workflow that will use those those secrets uh will look something like this. So no more hard-coded credentials, we just reference secrets that we've configured and the workflow just just works. And this can give you an impression that now all of your secrets are well secure because you cannot view those secrets once you configure them. You cannot view those secrets in the UI. It's just not possible. You can replace them but not view existing values. And also GitHub actions if you accidentally output those secrets as part of the workflow, GitHub actions will actually mask that output and will not allow you to leak accidentally leak those secrets in the workflows. Now having said that somebody might be viewing um GitHub documentation, GitHub hardening documentation and one of the warnings that they have there is any user with right access to your repository has read access to all the secrets configured in that repository. There's no more details than that. So it it's not clear why that's the case. So let's explore this a little bit more. So now imagine I I'm I'm an engineer. I have right access to to the repository and what I do is I create a new branch and I slightly modify this workflow that we had and instead of the actions that were originally in the workflow, I'll just try to exfiltrate the secrets. Now what I'll do next is go and basically run this workflow on the new branch that I created. So, I select my branch and I run the workflow. Let's give it half a minute and and see what happens. So, the workflow executed successfully and in the output we can see two things. Firstly, as we expected, if we just try to output the secrets, GitHub actions did mask those values. But if you do simple transformation like B 64 encode uh the values in this case, you now have access to to the secret values. So again to repeat this um any user with right access to your repository will have access to those secrets. Now, how how common is that that you have engineers or people in general that have right access to your repositories? That's pretty common. If you want somebody to contribute code to your repositories, they will have right access. And so one common way to deal with this and kind of strengthen the security model of of your source code is you will have branch protection rules and you will maybe require uh pull requests and approvals of those pull requests before any changes might might be merged into your repository. But my repository has branch protection rules on the default branch. And those branch protection rules do nothing to protect your secrets. and the secrets what's happening kind kind of underlying problem here is that firstly GitHub actions workflows are defined in the repository on which they run secondly secondly uh the the workflow definitions are taken from the branch on which the workflow is executed so if I create a new branch completely modify the workflow I can run the modified workflow on that branch that I've uh that I've created and thirdly Repository secrets are available to any workflow executions within that repository which means that that modified that compromised workflow definition will be able to access uh the secrets. So a combination of these three things ultimately give anybody with right access to the repo read access to the secrets. So now what can we do about this? There's a couple of ways of of dealing with this problem. One is using what is called environments. Sometimes they're called deployment environments in GitHub documentation. And environments also allow you to configure secrets or variables, but they also allow you to to configure protection rules on those environments. And protection rules allow you to control who or what can access those secrets, can use the environment and access those secrets. So let's take a look at that. We'll go into our repository settings again. And I already pre-created this environment that I called production. And I want to show you that there's multiple things that are available in terms of how you can secure the environments. One of the things that I that I have configured is basically I'm saying this environment can only be used by workflow that's executed on the main branch which is my default protected branch. No other branch will be able to use this environment. And now I can basically move those secrets that I had uh as repository secrets uh into this environment. So let me do that quickly. Uh by the way feel free to steal this secrets. All right. In a sense, this looks the same, but because of these protection rules, it it acts uh very differently. So, we can now go into the repository settings and delete our repository secrets. We don't need them anymore. They've been replaced by the environment secrets. So, this one is gone and this one is gone. And you can see that for visibility purposes, we see that we do have those those secrets that we've configured in the environment kind of listed here as potentially uh secrets that can be used. Once we've done all of that, our workflow is modified very slightly. It's basically the same workflow. The only change here is that I'm referring to that production environment in my workflow. And now I'm able to use those secrets when this workflow is run. So now let's try and and uh validate whether our our previous attack will actually work here. So I have yet another branch that has the malicious actions in the workflow. It still references this production environment. So I'm going to try and execute uh this workflow using my malicious branch. So that branch was called shady 2. And I'm essentially doing the same thing but on this uh new workflow that uses an environment. Give it half a minute again. And now our workflow execution failed because the protection rules of that production environment did not allow you the use of that environment on a on a nonprotected branch. So that's uh one way to deal with with the issue of secrets. Another way to deal with it is uh for thirdparty services that support this configuration. We can also use OIDC kind of instead of long lived credentials, we can use OIDC configuration uh trust configuration and shortlived credentials. So in this case there's kind of two parts to this. One part is modifying the workflow and basically completely removing the secrets and uh telling the AWS actions um action to or configure AWS credentials action to assume this role that we've preconfigured in AWS and in AWS um we will have an identity provider uh that we configure and we can create this role. So the name of this role was the same or the ARN of this role was the same as as in the workflow and in the trust policy we can specify what can assume this role and so in this case I wanted to demonstrate that a similar attack is actually possible even if you're using a shortlived OAC credentials. So in this case what I'm doing here on the trust policy is I'm essentially saying that any workflow on that repo that I'm using and this is the star here. This will essentially mean that any workflow in that repository will be able to assume this role. So let's validate that uh we can still attack do the same attack against now this shortlived credentials. So I'll take the same modified workflow that uses OADC um and uh again I'll slightly modify it uh to output the credentials. In this case uh we also need the session token because those are the credentials that we'll get from assuming the role. And so let's try to run uh this modified workflow. So, it's our uh fourth workflow and that uh sort of malicious branch was called shady 3 and we're going to run it and see what happens. So that execution was also successful and this was because of the mis misconfigured trust on on the role uh not because oidc in itself has any kind of drawbacks but we're still able to excfiltrate the credentials um and to fix this to fix this we need to modify the the role and we need to modify the trust policy in the role and we actually have two options here. One is to just specify in in that sub uh in the condition on the sub subject claim that we only allow this role to be assumed from this specific branch in this case the main branch. And so if we try to use this this role I called uh test GitHub actions ro three. So uh if we try to uh use that I'm going to show you quickly the uh that kind of attack workflow. So that's the 3.1 uh branch. It's it's trying to assume that RO three that is now fixed. And if we try to run it, it it will ultimately fail. Uh we'll give it another second. So the workflow is still running uh but ultimately it's trying to assume the role but failing and and trying to retry it and but it will ultimately fail after a number of attempts. Now the thing is you can actually combine the environments and protections that the environments give you with the OIDC configurations and what that looks like and in fact if you are using environments you have to configure the trust policy in this way. So what that looks like is in the trust policy configuration I'm also now referring to the environment gith github environment that is able to assume this role and if the workflow is not using that environment it's going to fail to to assume the role and so the modified workflow uh will look something like this so in this case this part is the same I'm referring to that role two but I'm also using this production environment uh in in the workflow definition. So if we try to now uh we can try to attack uh this workflow uh using a modified a workflow modified in a branch as well to exfiltrate the credentials. So now we're using that role too. And if we uh run that uh it's going to fail again. Let's see the result of that. So this failed again and this failed because our environment protection rules basically prevented the run of this workflow on on the unapproved uh unprotected branch. Another thing that we could try to do is sort of trick the the role assumption part uh uh and basically use the same workflow but now I'm assuming the right role but I've removed the the reference to that uh environment and see if this trick is going to work. When we run that it's going to fail again but it's going to fail in a different way. So it's not going to give us an error here because of the environment uh protection rules but it will give it will basically time out trying to assume the role because the role assumption is going to fail because of the role trust policy configuration. So let's quickly see that. So it's running. It's going to try to assume the role. I'm going to show you that it's kind of trying and retrying and ultimately this process is going to fail and and the whole workflow is going to fail. And just to show you that one of the earlier ones that was trying multiple times um has failed. All right. Uh so to the conclusions um security model of GitHub action secrets is not very intuitive and ultimately you have to remember that anybody with right access to your repo has access to those secrets. My personal recommendation never use them. You can use we can use uh deployment environments or environments uh to further secure our secrets and then you can use the protection rules on the environments as a security mechanism that controls access to the secrets and you can actually combine that for both long live uh credentials and our ADC configurations and you use basically your environments as the control mechanism that's going to be enforced across whether you use secrets or you use OADC configurations uh regardless of of that scenario. Uh one thing that kind of is part of the picture but I haven't uh covered in detail is that um in a sense um it does matter what triggers are allowed on your workflows because the trigger configuration always comes from your main your default branch and whether you are allowed to execute the workflow or not at all depends on the triggers. In this case, I've used manual triggers and that's partially why I was able to execute the workflows manually on any branch that I pick. But ultimately the triggers matter and I I suggest that you review the triggers that you use uh and and the workflows that they apply to to kind of understand what can trigger those workflows and and ultimately because of that kind of bait and switch I can redefine the workflow in in a new branch whether that attack is possible at all regardless of kind of secrets configuration and whatnot. And the last thing I'll mention, what I didn't cover is kind of specific scenarios specific to open source where PRs are usually created from forks, not from the branches within repository. There are some nuances there and kind of the the secret injection works slightly different for for those PRs and and I didn't cover that at all and it has some nuances. That's all I have for today. I have some uh additional reading material if you want to learn a little bit more about this topic. So that's the QR code on the left and if you want to connect with me on LinkedIn, you can scan the QR code on the right. And thanks for your attention. questions. Um, great talk. What would you say the most effective log sources are for trying to detect these things to see when someone's trying to steal your secrets? Um, I don't know a good answer to that question. One source is definitely workflow runs because if you go into GitHub and you you can see like previous runs including the output that uh those runs had but also including what branch the run was executed on that may or may not be a good audit mechanism because and I don't know exactly what permissions are required but you can delete past runs and also I'm not sure what the retention uh is on the runs. I assume it's not forever. Uh but that's one thing that you could check to see what like past runs what the runs were executed on. I bet there's also some logs. I haven't looked into kind of audit logging within GitHub to see like what's exposed there, but um my guess would be there there's definitely something should be there. I'll ask a question around um environments. Um I haven't actually use them all that much but like what are the maybe downsides of using that or is it going to be um the recommendation is to use kind of environments uh maybe other protections like you mentioned OIDC restrictions and all that sort of stuff about branch protections. Uh where can environments maybe go wrong? Yeah, I don't think there's any downsides to using environments. Um, as far as I can tell, they can provide the same things as repository secrets. You can configure variables, you can configure secrets, both things. And you have the benefit of um, being able to control what has access to them. Um, what those protections can be depend on your license. So for example in GitHub enterprise uh you can configure basically manual approvals on any runs that use specific environments and that's like if that's what you want to do you can do that. Uh but at a minimum it it will allow you to basically repurpose your branch protection mechanisms and apply something that's already fairly well understood and kind of intuitive to a lot of engineers. apply similar mechanisms to your environment protections and it ultimately good understanding of what's happening within your repo configuration. And I think it like clear understanding BR is good for security always and so sorry to clarify that too um when you're using for example uh OIDC trust policy in AWS y you have to use environments basically everywhere or like can you use the environment uh restriction along with the branch like restriction for example you can I think like the way you you're fairly flexible on the way how you configure the roles. So you can do both in the roles. What I meant when I said that you have to use environments is that if the workflow runs refers to an environment that subject will always have environment and name not the branch reference in the subject. So you if you if you are using the environments you have to have at least the the the statement or the condition in the trust policy that will kind of account for that and will refer to the environment as well. Cool. Thank you. All right. Let's give it up. All right. Thank you.