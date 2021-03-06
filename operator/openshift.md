# RedHat Operator Release Steps

## Summary

We presently still use the v1beta1 CRD. At some point we need to convert to the v1 CRD. However thsi CRD is too large for operator-registry (it converts CRD to configmap and it hits configmap limit it seems). We therefore might need to move forward with just v1 version for the v1 CRD and remove v1alpha2 and v1alpha3 versions of the SeldonDeployment CRD. See https://github.com/operator-framework/operator-registry/issues/385

There are also fixes in crd and crd_v1 configs for https://github.com/kubernetes/kubernetes/issues/91395 under a patch called protocol.yaml

We also remove the `owned` versions for v1alpha2 and v1alpha3 using `hack/create_graph_openapi_schema.py` to fix community test lint failures. This maybe actually be an issue in `operator-courier verify`.

## Update

Login to quay.io/seldon/seldon-operator

```bash
make recreate_bundle
make update_packagemanifests
make create_bundle_image
make push_bundle_image
make validate_bundle_image
make opm_index
make opm_push
```

## Scorecard

```bash
kind create cluster
```

Run scorecard

```bash
make scorecard
```

## Tests

Run [kind cluster tests](./openshift/tests/README.md). k8s >= 1.16.

Run on an openshift cluster. Openshift >= 4.3.

## Community and Upstream Operators

Will need to be run in release branch

Create a fork of https://github.com/operator-framework/community-operators

Create a PR for community operator

```
COMMUNITY_OPERATORS_FOLDER=~/work/seldon-core/redhat/community-operators

cp -r packagemanifests/1.3.0 ${COMMUNITY_OPERATORS_FOLDER}/community-operators/seldon-operator
cp packagemanifests/seldon-operator.package.yaml ${COMMUNITY_OPERATORS_FOLDER}/community-operators/seldon-operator
```

Run tests

```
cd ${COMMUNITY_OPERATORS_FOLDER}
make operator.test KUBE_VER=""  OP_PATH=community-operators/seldon-operator
```

Add new folder and changed package yaml to a PR. Ensure you sign the commit.

```
git commit -s -m "Update Seldon Community Operator to 1.2.2"
```

Push and create PR.

Do the same for the upstream communit operators

```
COMMUNITY_OPERATORS_FOLDER=~/work/seldon-core/redhat/community-operators

cp -r packagemanifests/1.3.0 ${COMMUNITY_OPERATORS_FOLDER}/community-operators/seldon-operator
cp packagemanifests/seldon-operator.package.yaml ${COMMUNITY_OPERATORS_FOLDER}/community-operators/seldon-operator
```

Run tests

```
cd ${COMMUNITY_OPERATORS_FOLDER}
make operator.test KUBE_VER=""  OP_PATH=upstream-community-operators/seldon-operator
```

## Certified Operators

Will need to be run in release branch.

Create new package

```
make create_certified_bundle
```

```
make build_certified_bundle
```


Push all images to redhat. requires download of passwords from 1password to `~/.config/seldon/seldon-core/redhat-image-passwords.sh`

```
cd {project_base_folder}/marketplaces/redhat
python scan-images.py
```

After these are finished (approx 1.5 hours) you will need to manually publish images on https://connect.redhat.com/project/5892531/images


Test as above for openshift but using the new catalog source. TODO: Needs creation of this catalogsource file in tests folder.


Push bundle image to scanning and tests. Also needs passwords.

```
make bundle_certified_push:
```


Publish image for final step to release new version of operator.
