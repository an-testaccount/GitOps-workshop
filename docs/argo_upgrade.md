
# Test - ArgoCD upgrade

But... Will it actually work?

To test it out we will let ArgoCD upgrate itself!
The installed version is v1.8.6 but v1.8.7 is out... let's upgrade!

Have a look at the `argocd_deploy/kustomization.yaml` file... Can you figure out how to do it?

Apply the change by committing and pushing to `master`, no need to use the docker command line anymore.
After a couple of minutes you should see the new version deployed!
