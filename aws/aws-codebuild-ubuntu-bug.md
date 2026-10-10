# AWS CodeBuild Bug with Ubuntu v26.04

In my AWS CodePipeline, AWS Codebuild builds a docker image that uses ubuntu as a base image.

Until few months ago, AWS CodeBuild was building the image without any error. However, in early September of 2026, it started throwing an error.

My dockerfile had a `RUN` command that installed and executed `tar`. But somehow, `tar` was not working properly.

## Error

```txt
tar: ./path/to/archived/file: Cannot open: Function not implemented
tar: ./path/to/archived/file: Cannot mkdir: Function not implemented
```

## Troubleshooting

Tweaking the CodeBuild environment such as using elevated proteges, using latest Amazon Linux version etc. didn't help.

## Cause

The cause turned out to be originating from `ubuntu:26.04` base image. Apparently, in ubuntu base image there was some kind update/change that was not compatible with AWS CodeBuild yet.

## Solution

Downgrading to older versions of the ubuntu such as 24.04 solved the issue. Since it is LTS version, it is supported until 2029.
