# SubsequentActionHelper

This is a helper action that can be used to create a subsequent action for a given action in the same repo.

* This is mainly a workaround because the 'uses' is not able to be dynamically set which makes it difficult chaining actions in the same repo more or less impossible. Without checking the repository out again.

## Table of Contents
- [SubsequentActionHelper](#subsequentactionhelper)
  - [Table of Contents](#table-of-contents)
  - [Inputs](#inputs)
  - [Usage](#usage)
  - [How it works](#how-it-works)
  - [Private Repository Tokens](#private-repository-tokens)

## Inputs
- Required
    - `repository`: The repository of the action that you want to create a subsequent action for.
    - `action-repo-clone-path`: The path where to clone the repository to.
- Optional
    - `private-repo-token`: The token to use to clone the repository. (Not required in the same repository of the action itself it - if you want to use the same ref as the action itself.)

## Usage
Public Repository:
```yaml
# In your custom action
- name: Clone the repository
  uses: PythonGabe/SubsequentActionHelper@v1
  with:
    repository: 'org/action-repo'
    action-repo-clone-path: './action-repo'

- name: Run another action in from that repository
  uses: ./action-repo/other-action # the important part is the path to the action needs to be the same as the path in the action-repo-clone-path
```

Private Repository:
```yaml
# In your custom action
- name: Clone the repository
  uses: PythonGabe/SubsequentActionHelper@v1
  with:
    repository: 'org/action-repo'
    action-repo-clone-path: './action-repo'
    private-repo-token: ${{ secrets.PRIVATE_REPO_TOKEN }}

- name: Run another action in from that repository
  uses: ./action-repo/other-action # the important part is the path to the action needs to be the same as the path in the action-repo-clone-path
```

## How it works
1. It will do a check to determine if the action is running in the same repository as your action or if it is being used in a different repository.
2. It will then put the repository in the 'action-repo-clone-path' and then you can use the action from that path.
    - If the action is in the same repository as the action itself and the action ref is the same as the action itself it will not clone the repository instead copy the contents of the repository to the 'action-repo-clone-path'. ** If the action ref is different it will clone the repository to the 'action-repo-clone-path' **
    - If it is being called in a different repository it will clone the repository to the 'action-repo-clone-path'
3. It adds a action-ref.txt with the ref of the action that is being used to the clone/copt to 'action-repo-clone-path'
    - This makes it so if in other action calls if there is a different ref it will replace the contents of the 'action-repo-clone-path' with the new ref. This allows for the actions to have different refs and still work.

## Private Repository Tokens
If you are using a private repository you will need to provide a token to clone the repository. This token should be stored in the secrets of the repository and then passed in as a input to the action.
- For organizations - its best to create a organization secret and then use that secret in the action, so all your repositories can use the same token.
- The token will need access to the action repository.


## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details
```