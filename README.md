# Architecture of My Personal Site

I wanted to create a site that was hosted in AWS, but was low cost.  I use C# daily, so it was my first option.  However, if I wanted to have a .Net MVC application, I would need a server to host it.

In the past, I've hosted sites using my own servers, which would be considered "on-premises".  I wasn't interested in doing that, so I turned to AWS and EC2.  Running EC2 is inexpensive, but knowing that this site wouldn't get a lot of traffic I chose to host on S3 instead.

S3 can serve static file assets really well.  This lead me to choose a JavaScript solution, instead of a .Net Mvc solution.  I initially created a pretty basic page, but the UI was not the best.  In parallel, I have been hearing and reading more about React and I wanted to test it out.  I started with a template [here](https://github.com/tbakerx/react-resume-template) and modified it to fit my needs.

Having made the necessary changes, I built out a deployment pipeline, so that if I ever need to make more changes, all I have to do is make my changes and check my code into a source repository.

# Architecture Diagram
![Diagram](https://github.com/ggkilmon/PersonalSite/blob/master/Diagram.png "Diagram")

Each time I check my code into CodeCommit, it kicks off a build, configured in CodeBuild.  Once CodeBuild succeeds, the CodePipeline deploys to the S3 bucket.

A user browsing to the url https://www.grahamkilmon.com will be directed to CloudFront, which passes through the requested assets from the S3 bucket.  My S3 bucket is not publicly available.  I could tighten control of assets flowing out of the S3 bucket by quickly modifying the behaviors in the CloudFront distribution.