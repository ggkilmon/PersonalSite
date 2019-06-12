# Architecture of My Personal Site

I wanted to create a site that was hosted in AWS, but was low cost.  I use C# daily, so it was my first option.  However, if I wanted to have a .Net MVC application, I would need a server to host it.

In the past, I've hosted sites using my own servers, which would be considered "on-premises".  I wasn't interested in doing that, so I turned to AWS and EC2.  Running EC2 is inexpensive, but knowing that this site wouldn't get a lot of traffic I chose to host on S3 instead.

S3 can serve static file assets really well.  This lead me to choose a JavaScript solution, instead of a .Net Mvc solution.  I initially created a pretty basic page, but the UI was not the best.  In parallel, I have been hearing and reading more about React and I wanted to test it out.  I started with a template [here](https://github.com/tbakerx/react-resume-template) and modified it to fit my needs.

Having made the necessary changes, I built out a deployment pipeline, so that if I ever need to make more changes, all I have to do is make my changes and check my code into a source repository.

# Architecture Diagram
![Diagram](https://github.com/ggkilmon/PersonalSite/blob/master/Diagram.png "Diagram")

Each time I check my code into CodeCommit, it kicks off a build, configured in CodeBuild.  Once CodeBuild succeeds, the CodePipeline deploys to the S3 bucket.

A user browsing to the url https://www.grahamkilmon.com will be directed to CloudFront, which passes through the requested assets from the S3 bucket.  My S3 bucket is not publicly available.  I could tighten control of assets flowing out of the S3 bucket by quickly modifying the behaviors in the CloudFront distribution.

## CloudFront
I configured my domain registrar's CNAME records * and www to point at my CloudFront distribution address (Example: *abc123*.cloudfront.net). 

My CloudFront configuration is fairly basic.  I specified my CNAMEs www.grahamkilmon.com and grahamkilmon.com.  I also specified the default root object as index.html.  This is the object first requested when hitting my domain.  An origin is specified that points at the S3 bucket that holds my static assets.  The default behavior is specified for all assets to redirect HTTP to HTTPS.

## CodeBuild
I initially had this configured to use a base image with nodeJS version 6, but this proved to be a bad idea.  I updated the base image to aws/codebuild/standard:2.0.  Its listening for check in events on the CodeCommit repository for the site.  The site itself has a buildspec.yml file which specifies the build actions, specified below.

```yaml
version: 0.2 
 
phases: 
  install: 
    runtime-versions: 
      nodejs: 10 
  pre_build: 
    commands: 
      - npm install -g npm@latest 
      - npm install 
  build: 
    commands: 
      - echo Build started on `date` 
      - npm run build 
  post_build: 
    commands: 
      - echo Build completed on `date` 
artifacts: 
  files: 
    - '**/*' 
  base-directory: 'build' 
```

I have 4 phases specified: install, pre_build, build, and post_build.

### install
This phase lets CodeBuild know that I want version 10 of nodeJS.

### pre_build
This phase lists commands that get run before the actual build.  In this case I update npm to its latest version and install all of the packages necessary to build this React app.

### build
This phase does the actually compilation/transpilation of the React site.  React puts all output files into a subdirectory called 'build'.  These are the files I ultimately want in my S3 bucket to serve to visitors.

### post_build
This doesn't do anything useful, but I put it in there for fun.  Post build things like copying files around could be done here if needed.

### Artifacts
The artifacts section is the output of this build.  As I said above, the contents of the build directory is all I want in my S3 bucket, so I specify the base directory, 'build', and all files and subdirectories in that base directory.

## CodePipeline
The pipeline zips the results of the CodeBuild phase and extracts and copies the files to the S3 bucket.  Once complete, updates are in the site.

## Notes
One thing to note with the above design, since the files are served via CloudFront, they are cached on various Amazon Edge nodes.  The TTL configured in CloudFront will eventually invalidate the cache on thoes nodes and the updates will eventually be served.  Since this is a personal site, I'm ok with that.  If I needed things to move to production quicker, I would adjust the TTL to a lower value and/or invalidate the offending assets within CloudFront itself.