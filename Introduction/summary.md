# Introduction

**Video Link**: [Watch on YouTube](https://www.youtube.com/watch?v=p8PZiqXoVTc)

- **Author**: Conference Organizers
- **Talk Type**: Opening/Introduction

## Summary

The opening introduction to Forward CloudSec 2025 sets the stage for the conference theme "Living on the Edge," reflecting how cloud computing has evolved from centralized infrastructure to distributed, edge-based architectures. The speaker welcomes attendees to a community-driven conference focused on emerging cloud security challenges including AI, GPUs, agent architectures, and new AWS services like Resource Control Policies.

## Key Points

- Conference theme "Living on the Edge" reflects the distributed nature of modern cloud computing
- 40 speakers presenting on bleeding-edge cloud security topics
- Five AI-focused talks addressing agent architectures, token theft, and cost control
- New AWS Resource Control Policies (RCPs) released in November 2024
- Data perimeters and data sharing challenges increase with AI data scraping
- Cloud security industry facing consolidation and cost pressures
- Community conference emphasizing collaboration and knowledge sharing
- 300+ independent cloud security practitioners attending

## Technical Details

**Conference Theme Context:**
- Evolution from centralized cloud to distributed edge computing
- New resource types requiring security consideration
- GPU security challenges with AI workloads
- Agent architecture security implications
- Token management and cost control in AI systems

**Emerging Technologies Covered:**
- **AI and Machine Learning**: Five dedicated talks on AI security challenges
- **GPU Computing**: Security considerations for graphics processing workloads
- **Agent Architectures**: New paradigms requiring security framework development
- **Resource Control Policies**: Recently released AWS service for data perimeter control
- **Data Perimeters**: Enhanced data protection and sharing controls

**Industry Changes:**
- Cloud security startup acquisition exceeding $30 billion in 2024
- Pressure on early-stage engineering careers similar to full-stack development trends
- M&A environment affecting cloud security startups and research teams
- Global regulatory and political impacts on cloud security practices
- Threat actor evolution enabling and limiting security responses

**Conference Infrastructure:**
- Community-driven event dependent on participant engagement
- Over 1,000 talk reviews conducted by organizers and volunteers
- Hundreds of volunteer hours for event preparation
- Code of conduct enforcement for respectful environment
- Open source project support through Granted team donation

**Sponsorship Structure:**
- **Gold Sponsors**: Permiso, Tamune, DataDog, Run Reveal
- **Silver Sponsors**: TraceBit, Sweet Security, Cydig, Whiz, Sunrise Security
- **Bronze Sponsors**: Red Canary, Chaser, Rock Steady, Hannibite, Clear Vector, Trust on Cloud
- **Lunch Sponsors**: AWS Vulnerability Disclosure Program, Netflix Cloud Security Podcast
- 30 personal sponsors supporting low ticket prices and scholarships
- 8 scholarship attendees supported by community contributions

**Speaker Support:**
- First-time offering of speaker honoraria
- Support for speakers regardless of employer travel funding
- Accommodation for international speakers facing travel challenges
- Recognition of global conditions affecting attendance

**Event Logistics:**
- Slack community integration with open invitations
- Birds of Feather sessions organized by participant interest
- Lunch service outside main presentation rooms
- T-shirt pickup during breaks
- After-party coordination through sponsors
- Real-time schedule updates on conference website

**Future Events:**
- Forward CloudSec EU in Berlin, September 2025
- Forward CloudSec North America 2026 (location TBD)
- Continued global partnership despite travel and visa challenges
- State of the Union closing talk by founder Scott

**Community Values:**
- Recognition of global challenges beyond cloud security
- Support for international colleagues facing travel restrictions
- Emphasis on safety, peace, and global partnership
- Commitment to inclusive participation across geographic boundaries

## Full Transcript

So our theme this year is living on the edge. Uh and it's because when we compare to when we started in 2020, people in 2025 have this very different view of what cloud means. It's not this central thing out in the ether. It's kind of distributed all around us. So we have these brand new bleeding edges that we need to play with and watch out for. So we've got all new types of resources that we're trying to secure. So, thanks to Matthew Braun for uh joining to teach us about GPUs. We've got five different AI talks this year. I think we're all struggling with uh agent architectures and what that's going to mean. Token theft, cost control. Uh I guess that's not a new problem. Uh resource control policies, which I had to look up. We've been talking about data parameters and forward cloud sec for several years. Um but RCPs just came out in November. So they give us powerful tools to create data parameters which is great but um you know definitely want to see Anil Emodia and Ben Joyce's talk there uh but the demands for data lakes data sharing data lockdown as uh AI scrapes everything uh is only increasing and that's the technology side of things. So if we look out over like the edge of tomorrow, the changes in our industry and in our community are not just technical. It's like what business isn't a part of the cloud now? And you would think this would be great for us as security practitioners, right? This is a year that a repeat forward cloud seconsor sold for over $30 billion. Hi. Um but this is an existential change for us, right? Is cloud security just security now or even just technology? uh we cloud security people have yet to see I think the same significant impacts on the early stage engineering careers that our colleagues who do full stack or mobile development have seen but it's coming the same M&A environment cost pressures uh and AI tech that makes our lives exciting uh also puts pressure on the cloud security startups that form a part of this community the research teams that are doing research and the practitioners to be ever more efficient and ever more focused and of Of course, outside of that, outside of engineering, the global regulatory and political environment, the threat actors, these alternately enable and limit us. Uh, and we can hope, if we're lucky, that the choices we make can have a measure of positive impact there. I will say that I have been doing cloud stuff for a decade or so now, and I think this is probably the hardest year to predict that since I've been doing it. So, here we are in the mile high city. Uh, hopefully looking out over the edge of an uncertain horizon. But this is exactly why we're here. It's time to figure it out together. So, if you've never been here before, quick reminder. This is a community conference. It is what you It is what we make of it together. It is the questions we ask in person, on Slack, in birds of a feather sessions, out in the room. Uh my fellow blue shirts wearing these shirts. We are here to assist with questions. If necessary, help ensure our code of conduct makes for a respectful environment. Uh I want to start with a huge thanks to all of my fellow organizers, the dozens of volunteers. We did over a thousand reviews again this year, hundreds of hours of prep, uh 40 speakers who are here to educate and guide us. That forms the skeleton of the event. Uh I want to say thanks to Chris Norman and the granted team. Uh so they donated and are maintaining forward cloudex first open source project. That's pretty cool. Um and now we have granted fans. Uh of course I have to thank the sponsors. Uh this event doesn't exist without them. Uh certainly not in the same form. So big thanks to our gold sponsors, Permiso, Tamune, Data Dog, Run Reveal to our silver sponsors, Trace Bit, Sweet Security, Cydig, Whiz, and Sunrise Security. to our bronze sponsors, Red Canary, Chaser, Rock Steady, Hannibite, Clear Vector, Trust on Cloud, Lunch Sponsors, uh AWS's vulnerability disclosure program, uh wants to talk to you, uh as well as Netflix, uh the cloud security podcast. Uh this year we had 30 personal sponsors, uh who made an extra contribution with their ticket, helps keep the ticket prices low for everyone else, helps support the eight scholarship attendees who are here. And I see hands right in the front row. welcome scholarship attendees. Uh and that also helps us offer for the first time honoraria for speakers. Uh which helps us get the best talks regardless of whether their employer is playing for their flight or not. Uh last thing also I want to thank our colleagues overseas who run the EU conference and this year especially I want to thank the participants and speakers who wanted to be there but couldn't make it uh due to the changing world conditions. So that's visa challenges in in a number of cases and of course global violence. Uh we are here to talk about cloud security but we know this isn't the most important thing going on. So uh for those of you who could not make it sending all of you our best wishes for safety peace uh return to global partnership. We will see you on Slack. We will see you on social media. We will see you on streaming. Hopefully we will see you in Berlin in September and back next year for Forward CloudSc North America 2026 wherever that may be. Uh if you want to learn more about what it takes to create and maintain a community like this, uh our founder Scott is bringing back his closing state of the union talk that's going to be 3:30 tomorrow right here in this room. So uh let's kick it off, right? Living on the edge for Cloud Tech 2025. It's supposed to be more exhilarating than easy, right? Yes. Yes. Maybe you think there's something wrong with the world today, but if Chicken Little tells you that the sky is falling, I'm here to say you should tell him no or let it go, right? Sure, I've spent enough of my life debugging Terraform and I am that it sure ain't no surprise that a lot of our conversations here are about how complication aggravation is getting to you. But that's okay. When we run into each other in the hallway, tell me what you think about your situation. We can work together to make sure we're seeing things in a different way and prevent meltdown in the sky or cloud. I think meltdown in the cloud. Yeah. Cool. Okay. Look, we didn't book Aerosmith. Too much to Chris's great disappointment. I'm not singing. Uh Forest is not here. You just missed Metallica over at Mile High Stadium last night. I'm sure that was super fun. Uh but we're going to have fun here, too. We have talks. We have birds of feather sessions. We have parties. And most importantly, we have over 300 of the best independent cloud security practitioners in the world all in one place. Uh so a few announcements quickly before we get started. If you have not joined our Slack yet, invites are open. That QR code is right up there on the screen. If you don't see it right now, one of the blue shirts are happy to get it to you uh in the break. Um lunch will be at noon outside these doors. um feel free to pick it up, bring it back in or take it in any of the birds of a feather sessions or any of the track rooms. The only thing we ask is that when you enter and exit the rooms, please leave through the back doors so it's not too noisy in here. Uh birds of a feather sessions, we just do these kind of a little bit loosely. Uh folks have suggested some ideas on the boards next to registration. If there's a topic that's of interest to you, grab a sticker, put a sticker next to it. Uh uh Lee, one of our organizers, is going to be helping coordinate and pick the most popular ones, and we'll organize those at the end of the day today and tomorrow. Uh if you ordered a t-shirt, uh you can pick up t-shirts from registration during breaks and talks. Uh there are after parties. I don't know many details about them, but our sponsors do. Um and our sponsors would like nothing more than to talk to you uh and invite you to uh talk to them more after the event. So, uh, please say hello and and and maybe we'll, uh, get a drink together this afternoon. Uh, last, uh, we are continuing to make schedule changes to accommodate travel travel blips. Uh, most up-to-date schedule is always going to be the one on our website. Uh, so that is it. Our first talk starts in I think 10 minutes, something like that, right here, track two across the hall. Welcome. I'll see you there.