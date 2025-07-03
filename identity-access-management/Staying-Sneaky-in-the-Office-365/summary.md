# Staying Sneaky in the Office (365)

**Video Link**: [Watch on YouTube](https://www.youtube.com/watch?v=l5lpIF_QZCE)

- **Author**: Christian Philipov (Reverent Security)
- **Talk Type**: Security

## Summary

This presentation reveals three new SharePoint attack techniques that allow attackers to enumerate content, exchange tokens, and download files while avoiding Microsoft Graph API logging. The research demonstrates methods to stay stealthy in Office 365 environments by leveraging lesser-known SharePoint APIs and misconfigurations, providing both offensive capabilities and defensive insights for security teams.

## Key Points

- Three novel SharePoint attack techniques discovered that avoid Microsoft Graph API detection
- SharePoint enumeration through undocumented APIs bypasses traditional monitoring
- Token exchange capabilities allow pivoting from SharePoint access to broader Graph permissions
- Pre-authentication download URLs bypass sharing restrictions and network controls
- Detection challenges due to SharePoint natural noise and legitimate user activity patterns
- Simple PowerShell commands can prevent most dangerous attack (pre-authentication feature)

## Technical Details

**Research Objectives:**
- Avoid Microsoft Graph API usage to evade enhanced logging and detection
- Discover quick SharePoint enumeration methods for offensive engagements  
- Identify new attack primitives and defensive techniques
- Focus on lesser-explored APIs and undocumented endpoints

**Attack Technique 1: SharePoint Enumeration**

*API Endpoint:*
- Uses relative URL parameter (A1) to specify SharePoint library path
- Requires minimal request body: `{"parameters": {"__metadata": {"type": "SP.RenderListDataParameters"}, "RequiredFields": true}}`
- Returns comprehensive JSON response with file metadata and full paths

*Required Authentication:*
- Only needs FedAuth, RTFA, and SIM cookies from user session
- No Microsoft Graph token required
- Avoids Graph activity logging entirely

*Response Data:*
- Massive JSON blob containing detailed file information
- Key field: "FileRef" provides complete file paths within SharePoint library
- Can be processed with jq to extract file listings efficiently

*Detection Challenges:*
- Activity appears in Office 365 activity logs and unified audit logs
- Identical behavior to legitimate SharePoint search functionality
- High noise level makes reliable detection extremely difficult
- Cannot distinguish malicious enumeration from normal user behavior

**Attack Technique 2: SharePoint Token Exchange**

*Endpoint Structure:*
- Located at: `/sites/{site-name}/_api/SP.OAuth.Token/Acquire`
- Accepts resource parameter in request body to specify target resource
- Returns access token with associated scopes for requested resource

*Token Capabilities:*
- Primarily read-only permissions suitable for enumeration
- Notable dangerous scopes include:
  - Sites.Selected: Access to specific SharePoint sites
  - User.Invite.All: Ability to send guest user invitations
  - User.RevokeSessions.All: Can terminate other users active sessions

*Supported Resources:*
- SharePoint instances within the tenant
- Microsoft Graph API (limited scope)
- Azure AD Graph API (deprecated)
- Specific Microsoft 365 services

*Security Implications:*
- Enables pivoting from SharePoint access to broader Office 365 enumeration
- Bypasses some tenant restrictions on guest user invitations
- Provides denial-of-service capabilities through session revocation
- Allows token refresh without traditional OAuth flows

*Detection Issues:*
- Token refresh activity visible in non-interactive Entra logs
- Normal SharePoint operation includes automatic token refreshes
- Difficult to distinguish malicious from legitimate refresh patterns
- Would require behavioral analysis and baseline establishment

**Attack Technique 3: Pre-Authentication File Downloads**

*Discovery Process:*
- Standard file preview request: `/sites/{site}/Lists/{library}/AllItems.aspx?id={file-path}`
- Response contains two download URL fields:
  - `DownloadUrlNoAuth`: Standard authenticated download
  - `DownloadUrl`: Contains temp_auth parameter with special token

*Temp Authentication Token:*
- Format: `V1.{jwt-like-token}`
- Non-standard JWT structure with V1 prefix
- Functions similar to AWS pre-signed URLs
- Valid for 1 hour from generation
- Limited documentation references in SharePoint PowerShell modules

*Security Bypasses:*
- **Anonymous Sharing Restrictions**: Works regardless of tenant sharing policy settings
- **Network Location Controls**: Bypasses IP allowlisting and geographic restrictions
- **Authentication Requirements**: No user authentication needed during download
- **Audit Trail**: Downloads may not generate comprehensive audit logs

*Feature Status:*
- Currently being deprecated by Microsoft but still active
- Feature enabled by default in many tenants
- Can be checked with PowerShell: `Get-SPOTenant | Select-Object LegacyAuthProtocolsEnabled`
- Easy remediation: `Set-SPOTenant -LegacyAuthProtocolsEnabled $false`

**Token Analysis:**
- Partially decodable using base64 methods
- Contains SharePoint user identity information (not Entra identity)
- Includes token expiration data
- Some claims remain encoded or unclear
- Appears tied to specific user identity within SharePoint context

**MSRC Response:**
- Classified as low severity security issue
- Reasoning: Users with read access can typically export documents anyway
- Microsoft stance: "Jumping a security guard rail" rather than "security feature bypass"
- No architectural changes planned despite security implications

**Detection Strategies:**

*Limited Options:*
- Monitor Office 365 activity logs for file access patterns
- Baseline normal user behavior for anomaly detection
- Alert on unusual download frequencies or patterns
- Geographic analysis of access attempts

*Challenges:*
- SharePoint generates significant legitimate noise
- File access is normal user behavior
- Anonymous downloads may not log source IP or user identity
- Pre-authentication phase occurs before audit logging

**Defensive Recommendations:**

*Immediate Actions:*
- Check pre-authentication status: `Get-SPOTenant | Select-Object LegacyAuthProtocolsEnabled`
- Disable if not required: `Set-SPOTenant -LegacyAuthProtocolsEnabled $false`
- Test in non-production environment first
- No apparent legitimate business use cases for most organizations

*Ongoing Security:*
- Monitor SharePoint access patterns for anomalies
- Implement behavioral analytics for file access
- Regular audit of SharePoint sharing configurations
- Network monitoring for unusual download patterns

*Detection Development:*
- Focus on high-impact activities rather than enumeration noise
- Develop baselines for normal SharePoint usage patterns
- Consider correlation with other security events
- Monitor for rapid sequential file access across multiple libraries

**Research Implications:**
- SharePoint contains numerous undocumented APIs with security implications
- Traditional Graph API monitoring may miss SharePoint-specific attacks
- Legacy features present ongoing security risks despite deprecation
- Defense requires understanding of platform-specific behaviors and configurations

## Full Transcript

Hey everybody, welcome back. Welcome back from the break. Appreciate you showing up. Uh take a seat whenever you get a chance and we're going to go ahead and get started here. Um so this talk is going to be staying sneaky in the office 365. Uh just want to give a uh a thank you shout out to um a sponsor uh for this talk um is Tam Noon. Uh if you want to find out more information about Tamune, you can check out tamnoon.com or you could also talk with uh some of the folks that are out in the lobby if you're interested in hearing more about what they have to offer. Um in addition to that, um any questions that we have for this talk, we'll go ahead and just wait till the end. I'll bring the microphone around if you've got a question. Also, we'll have a Slack thread that you can uh ask some questions there and we'll go through those. Uh without further ado, please welcome uh Christian uh Philippov for staying sneaky in the office 365. All right. Thank you. Thank you. All right, then. Uh well, let's get started. We'll we'll talk a bit about well what the talk is about. Uh we'll talk a bit what was the main aim of the research. uh then we'll go through the results and then get some general outcomes out of it and how to go about trying to defend your environment to try and detect some of these things. So pretty straightforward agenda uh al together. So let's kind of kick off. Um so who am I? I'm Chris Philipov, one of the principal security consultants at Reverse. Uh so I do a lot of cloud uh mostly enter and Azure uh as you might have probably guessed and generally enjoy breaking stuff. You know, sometimes I enjoy building stuff, but then I have to maintain it. So, I'm kind of much prefer breaking it because then somebody else has to deal with the after effects. So, why are we here? Well, recently and you'll find out more details about this at at the talk tomorrow from uh one of my well one of my colleagues Nick Jones and then Mo Hit Gupta who's presenting kind of his parts recorded about we had a massive kind of IT migration as we move to kind of uh being an independent business um from what we were previously with secure and so we had a lot of tasks at hand to migrate so obviously as I'm kind of working on a lot of the enterure migrations. Um, I do what everybody else that has, you know, a lot of deadlines and a lot of tasks on his plate. I get bored with it. And so, obviously, the next best thing was I figured out, you know what, I'm either way kind of working a lot with SharePoint right now to do this migration. So, might as well, you know, take advantage of this and let's kind of start figuring out how SharePoint actually works in a bit more detail. And so, you know, let's let's do something a bit more interesting, kind of switch over, switch gears a bit and get back into kind of the research mindset. So, I decided to be a bit more adult than usual about this with the research. And instead of just kind of doing ad hoc stuff, I decided to sit down and set some set some goals, right? So, I didn't have that much time and I wanted to keep it simple. So, I got some highle goals that I wanted to do for the research. I wanted to avoid using MSG graph as much as I can. So some of the talks already that we that have happened kind of the talk uh in track one with Tom Burn on kind of rebuilding road recon uh Microsoft graph and Microsoft are getting better at documenting a lot of the activities that happen against the graph endpoint and so that includes enumeration activities and so I kind of wanted to see okay you know are there any kind of APIs that we can take advantage of that won't really use graph that much so that we can use those instead Uh I also wanted to discover some quick ways or methods of enumerating kind of SharePoint that can be used on offensive engagements and uh ideally wanted to discover kind of some new attack primitives or defensive agent techniques off the back of it. And so all in all we combine all these and what did we ultimately get? Unknown endpoints or less explored APIs. Uh in my defense I know what some of you are probably saying. Does he not do anything else apart from finding uh you know undocumented endpoints? Yes, I do. But this was more interesting. So that's why that's why it's in the talk. So don't judge me. I'm not trying to steal Nick Fchett's crown on the finding undocumented APIs, you know. I'm just trying to carve out my little section of uh of the Azure and enter bits. So off the back of it, we found three new cool outcomes uh from the research which would be useful for somebody trying to target SharePoint and can be useful for us as defenders to try and detect and prevent some of them. Some of them we can prevent. Technically we can detect most of them. But you will see that unfortunately it's a bit complex to try to detect them. And mostly it's because of the lot of the noise that gets generated by SharePoint naturally. Uh as I'm sure all of you are aware and have you know interacted with SharePoint in one manner or another. Uh SharePoint it's pretty massive and widely used in a lot of enterprises. And so there is a fundamental layer of noise that you can kind of hide in for these actions. So enough about me teasing about what the actual research outcomes were. Let's just kind of start showing them. So the first one, SharePoint enumeration. So we start off with an API endpoint that uh is used to get lists of items from a provided kind of relative URL within a SharePoint site. So we can see that in the A1 parameter over here uh on the second uh on the second line where you provided kind of the path uh within a given SharePoint site and then you only need to provide this basic body of adding required fields equals true for it to actually work. And what you end up getting off the back of that is a nice kind of massive JSON blob uh that provides you with too many details frankly about the files that you're trying to find. But the one that you realistically care about is that nice file reference one which just tells you the full path of the files within that library folder. So obviously you do that you collect a lot of your data about the SharePoint library and then you can pass it through something like JQ grab the relevant relevant kind of reference there and then bam now you've got the list of all the all the items in that SharePoint library. So easy peasy you know that's kind of gets us started. So this is this is good because that way we can avoid using the equivalent graph enumeration endpoint which you can see over there where you provided the relevant kind of site ID and the kind of what's called the drive ID for these unique resources and that way you could typically kind of enumerate any items that you as a user have access to. So for this one we basically only need uh these couple of cookies the fed the R RTFA and SIM cookies uh that you can get from a user endpoint and that way you can do this enumeration activity. So you don't necessarily need to compromise and get access to a proper graph um graph uh token. Uh there's no login in the graph activity logs for this one. Uh there are events that you can see in the office activity logs and kind of the unified audit logs. But like I kind of said a few minutes ago, the problem here is that this is an activity that happens very naturally when people just use the basic search functionality within SharePoint through the web browser. And so it is incredibly hard to properly use this as a single kind of detection metric because it is fundamentally noisy. The it just that's just how how it is. But okay, great. We managed to get kind of one objective down from the from the research topic. So, let's try and find some more interesting ones. SharePoint token exchanges. For the people that hope were at the other session in track one with Tom Burn kind of talking about road recon, you got a bit of an idea of how access tokens kind of and refresh tokens work within Entra and Azure. They slightly deviate from the typical kind of OOT spec uh of how they should be implemented in the sense that they allow you to use a refresh token to mint access tokens for various kind of resources and client ids in this case. Uh well typically a lot of that happens through the main kind of Microsoft endpoint which is login.microsoftonline.com and that's kind of primarily what you end up using to do a lot of these token exchanges. Uh however, SharePoint itself has its own equivalent kind of um access token exchanging method that exists within a given kind of SharePoint site. So you can see uh there's the slash sites then the name of the site and then this SP token acquire endpoint. the the main thing that you do is in the body you have you want to supply it with the resource uh that you want an access token for and then off the back of that you get a nice little request response uh that gives you the access token that you've requested and also conveniently it already tells you the types of scopes that you get from that access token. All in all um a lot of the scopes are kind of read only. So in that sense it is a very useful token for sure for enumeration activities cuz this way you can basically pivot from access tokens you manage to get from SharePoint directly to start interacting with the graph itself. But there are a few of those permissions that can be a bit more dangerous and useful kind of for an attacker. You've got site selected so that could allow you access to particular sites kind of that the access token is allowed to talk to. Um another interesting one is you can do user invite all. So technically if there are restrictions in terms of uh only certain users are allowed to invite guest users to the tenant then this permission uh should be enough this the scope should be enough if you manage to get one of those users tokens to then send out guest invites to other uh accounts that you've got access to. and then kind of user revoke sessions all kind of a useful one from a den now service perspective if you want to start kind of booting people off their active sessions but yeah all in all it's not kind of a you know fully you know critical access token that you manage to get but it will definitely get you going from from an enumeration perspective like all in all it's a very simple and straightforward way of just asking for different resource URLs and then getting tokens from it now the Good thing about this is that the endpoint does have some limitations. Uh you can't just exchange tokens for arbitrary uh resources. These are the couple of resources that I managed to kind of confirm that the token exchanges works for. So you can get an access token for at least some of these resources including your obviously specific kind of SharePoint instance that you've got within the tenant. um the usage of the refresh token to kind of get new well getting these new access tokens is still visible uh to some extent in the non-interactive kind of enter logs. So there is still evidence of uh that activity happening in the tenant. However, this comes back to the same problem we had with the first method which is this is activity that SharePoint does naturally in the browser as people are just using it. Every now and again it refreshes the tokens uh to get new ones. So every user using SharePoint will be doing this. So frankly uh unless an attacker is doing a significant amount of these uh refreshes to try to get new access tokens, it will be incredibly hard to make use of the audit logs available to actually build detection cases around this. Okay, great. from the initial kind of research goals that uh I've set up, you know, I played around with it with a few days, managed to get a couple of interesting kind of outcomes already. I started uh sitting down to document them and move back to actually doing some of the less interesting tasks. Uh but then obviously um that isn't exactly what ended up happening because we ended up finding one more interesting one just as I was kind of finishing up. So this is typically the request that you will see when you try to kind of open a file or preview a file within SharePoint. Pretty straightforward. You've got the path to the library. You've got this all items aspx file and then an ID that points to a specific kind of full path to the file that you're trying to get to. So in this case, flag1.txt. Okay, great. So I was playing around with this and checking out the response. Uh, one interesting thing that you encounter when going through the massive JSON blob, uh, this is this is just like half a megabyte worth of request by the way, which I think easily suggests why SharePoint can feel a bit sluggish if every single request is almost of that size. But anyway, um, we've got two specific fields that are interesting. We've got download URL and then download URL, no authentication. Now you would think that this is the dangerous one but not actually because this is the actual URL that you get when you interact with the file and try to download it naturally through SharePoint because what this ends up using is that this effectively pops this uh obviously is in the actual HTTP request and then it uses your access token to download the file and verify that you've got access to this and that you are allowed to access this and then downloads the file based on those checks. But the download URL one is interesting because I'm sure most of you have probably spotted. There's this nice URI parameter called temp temp authentication. And you know I think a natural question that probably a lot of you are asking right now that deal with Microsoft is what what what is this? Uh I've got no idea what this is. uh and that was exactly the same because this was the first time I ever encountered that uh properly especially with the fact that the token is ever so slightly different with this V1 kind of first part to it and despite the fact that the rest of the token seems to kind of follow as very standard kind of JWT token format that we're used to. So obviously uh finding this I was like okay let's try to find some more info about this you know what's what's what's there but unfortunately that wasn't particularly successful uh this seemed to be a very weird one where there wasn't really many references at all to this uh as a thing there were there are a couple references to temp off being used in the context of kind of one drive uh downloading uh but it it'slight slightly different. It uses a more standard kind of JWT structure there and is used more so kind of for modern interactions and kind of handling um kind of larger files or kind of downloading larger sequences of data and kind of handling that authentication part. But there wasn't really proper proper references to this kind of format within SharePoint. Eventually, we did find one reference uh after a bit of fiddling around and it was within the PowerShell module documentation uh for SharePoint online's kind of PowerShell module about getting the SPO tenant pre-auth within the environment. Luckily enough, somebody had decided to add a nice little note to it which explained or at least gave some context of what's actually happening. And you know, basically you can go and read through it, but for the most part, it effectively is kind of like a pre-signed URL uh for those familiar with AWS. And it generates this access token that is uh to be used for kind of more rich download scenarios uh to kind of handle downloading files uh without having to go through the typical authentication process of how a user would typically access stuff, right? Also of note that the feature is currently being deprecated, but it's still still live up until last evening. So I don't know how long that's been there and how long it's in the process of being deprecated, but uh anyhow the question I'm sure a couple of you are asking is okay great. You know what's the impact of this? Well, the practical impact is that we can download files from SharePoint. Uh we can download files from anywhere. Uh some of you that have kind of done SharePoint administration and management know that there is a setting in SharePoint that you can restrict who you can share files with. And most organizations, especially ones that are more kind of have access to sensitive data, default to only allowing users to share files within the same organization and disable anonymous link sharing. Right? This bypasses that. this uh this doesn't care if it's set up to uh disable any sort of anonymous sharing or public sharing. Um these links will work regardless for the 1 hour period that that token is valid for. There's an additional security control within SharePoint that's uh a bit more powerful which you can even restrict from a network location who can download whatever or in in general interact with SharePoint. And so if you're not from an allow listed IP in SharePoint, you just get fundamentally blocked from the service to the point where you can actually lock yourself out easily from SharePoint. And so Microsoft do make a clear clear point to just make sure that you add an IP that you do have access to cuz otherwise you have to raise a support ticket. This one bypasses that. It uh it doesn't do any of that checks. So even if the IP is not allow listed through that setting, you can still download it uh as as it worked. And so any member with the ability to read a file automatically generates these tokens in every single like request that's sent to kind of read or open that file. So good news using the the helpful kind of command lit that either way gave us a hint of how you know what's actually happening. We can use that to confirm what's the status of this uh functionality and feature within our tenant. So you can see in this case it is enabled because um you know we couldn't just label it enabled equals false but we have to go the double negative there. Um and the lucky thing is that at least from a prevention perspective we can also easily disable this by do by running only one commandlet I raise it to MSRC to try to see kind of what's the opinion and whether or not you know this is expected. Um, unfortunately not exactly the kind of the the what I expected from MSRC, but basically they determined it's a low severity issue because anybody with the ability to uh to read a document can find ways to export the document in some way or another, right? Which yeah, I can I can get that. Um, but the part that I particularly liked from the response is the last part, which is that we do not consider this a security feature bypass. Instead, we see this jumping a security guard rail. You know, some of you might ask, isn't this kind of the same difference? But, uh, I I don't know. I'll be honest. Uh, I'm still kind of wondering about that one. But, at least the lucky thing is it's easy to resolve. It's easy to check. So, hopefully most organizations can kind of quickly remediate this. So overall in conclusions we've got some cool new/ old uh defensive Asian techniques and kind of methods that we can use the SharePoint APIs to do kind of offensive activities. you know, there are some new approaches uh that we that there's definitely, you know, still some uh still some time to do like further research into. And I would definitely recommend anybody that's interested in this to kind of have a play themselves cuz I'm sure that there's probably a trove of other similar functionality that people can discover and it will be good fun for the community. We have a way to prevent or detect some of them. Mileage might vary. Uh, I'd say definitely still focus on kind of u the more high impactful stuff instead of necessarily kind of putting in significant amount of efforts here, but I will definitely recommend at least to look at that pre-authentication configuration within your tenant and probably disable that because at least from my Googling as I said basically no real references to this uh whatsoever at the moment and so I can't really see any legitimate use case of this within the tenant. So I think it's pretty safe to say you can probably disable it and it won't impact any workloads. Obviously test it in a test environment beforehand. Uh as much as I like to yolo and prod, you know, maybe maybe don't. But yeah, that's that's it for me. Nice work, Chris. Thank you. Um okay, so um any questions from the room here for Chris? I see I've got one over there. making my way. So you said you have ways to detect this. Are those documented as well that you can share? So the the the detection for the first two we kind of talked through in the sense that there are audit logs either in the office activity logs uh for for example file access and kind of that sort of file listing that would give you some hints if somebody's trying to enumerate SharePoint. However, because it's also what users genuinely do when they just visit it, it is a bit hard to make proper use of it. You can probably try to make some sort of baselining, but that's going to likely yield some false positives. So just keep that in mind. For the second one, for the token exchanges, you can look at kind of non-interactive like refreshes from a given user uh to getting kind of minting new access tokens. But again, because SharePoint does it naturally under the hood as well, very often that's also going to be significantly hard. I'd say even harder than the file enumeration stuff. The last one for the download and kind of pre-authentication stuff is also hard because essentially what you're going to be looking at in the audit logs is somebody kind of reading a file. And unfortunately with SharePoint that's kind of a pretty natural thing that everybody's going to be using. So you can maybe build some detections around it, but I'd say probably your better bet is to focus on trying to disable that feature all in all because then at least you can only you have a consistent method of accessing these files and you know that there is no way that you can anonymously kind of share these directly from SharePoint itself. uh but the actual act of somebody downloading that file from let's say some IP or if somebody has that link I couldn't see any logs of that within it because technically it happens before any authentication process whatsoever and so I don't think that the logs would actually have that available even in the office activity logs so you'd basically be relying on trying to guess if somebody's read the file and maybe exfiltrated the token from that and then used it to download files from somewhere outside of your kind of network control. Thanks. No, no problem. Awesome. Other questions? Yep. Uh thank you. Really good stuff. Just one question. Did you try decoding that token that was in the signed URL at all? I did. Uh it kind of decodes uh but not completely. Uh there's some parts of it which are just basic B 64, but the formatting of the token uh I need to sit down to kind of go through what they've got in terms of like some of the C classes that for decoding tokens and just see kind of what specific method to use for that one. the the answer is yes, but it's kind of uh incomplete of a decoding, but it definitely seems like it ties it to a given user identity uh that within a SharePoint user identity specifically, not not an enter one. Um you can kind of get the expiry, but uh other claims kind of of interest, not really at the moment. Some of it is just kind of garbage data, at least at the moment. Cool. Thank you. All right, awesome. Thanks for all the questions. If you've got any more questions, I would say reach out to Chris afterwards and and continue the conversation. Um, with that being said, let's give a final round of applause here for Chris. We appreciate it.