# SubsequentActionHelper

This is a helper action that can be used to create a subsequent action for a given action in the same repo.

* This is mainly a workaround because the 'uses' is not able to be dynamically set which makes it difficult chaining actions in the same repo more or less impossible. Without checking the repository out again.

## Table of Contents
- [SubsequentActionHelper](#subsequentactionhelper)
  - [Table of Contents](#table-of-contents)
  - [Inputs](#inputs)
  - [Usage Examples](#usage-examples)
    - [Public Repository:](#public-repository)
    - [Private Repository:](#private-repository)
    - [Multi Level Nesting](#multi-level-nesting)
  - [How it works](#how-it-works)
  - [Private Repository Tokens](#private-repository-tokens)

## Inputs
- Required
    - `repository`: The repository of the action that you want to create a subsequent action for.  You can hard code this or get it from the action it self.
    - `action-ref`: The ref of the action that you want to create a subsequent action for.
    - `action-repo-path`: The path where to clone the repository to. # This needs to be a relative path starting with "./"
      - Example: './custom/actions'
- Optional
    - `private-repo-token`: The token to use to clone the repository. (Not required in the same repository of the action itself it - if you want to use the same ref as the action itself.)

## Usage Examples
### Public Repository:
```yaml
name: "Your Custom Action"

inputs:
  action_repository:
    description: "Internal use only - do not set"
    required: true
    default: "${{ github.action_repository }}"
  action_ref:
    description: "Internal use only - do not set"
    required: true
    default: "${{ github.action_ref }}"

runs:
  using: "composite"
  steps:
    - name: Checkout Action Repo - for Subsequent Actions
      uses: PythonGabe/SubsequentActionHelper@1.0.0
      with:
        repository: "${{ inputs.action_repository }}" # Or you can hard code the repository - DO not put "${{ github.action_repository }}" directly in the uses - it will then pull this project repository.
        action-ref: "${{ inputs.action_ref }}"
        action-repo-path: "./custom/actions"
```

### Private Repository:
```yaml
name: "Your Custom Action"

inputs:
  github_actions_token:
    description: "The token to use to clone the repository."
    required: false
  action_repository:
    description: "Internal use only - do not set"
    required: true
    default: "${{ github.action_repository }}"
  action_ref:
    description: "Internal use only - do not set"
    required: true
    default: "${{ github.action_ref }}"

runs:
  using: "composite"
  steps:
    - name: Checkout Action Repo - for Subsequent Actions
      uses: PythonGabe/SubsequentActionHelper@1.0.0
      with:
        repository: "${{ inputs.action_repository }}" # Or you can hard code the repository - DO not put "${{ github.action_repository }}" directly in the uses - it will then pull this project repository.
        action-ref: "${{ inputs.action_ref }}"
        action-repo-path: "./custom/actions" 
        private-repo-token: ${{ inputs.github_actions_token }}
    
    - name: Run Subsequent Action
      uses: "./custom/actions/your-other-action-in-the-repo"
      with:
        # Your inputs
```

### Multi Level Nesting
- The nested is optional as the action itself will not copy/clone again if the action is in the same repository and the ref is the same.
- But I like it since it speeds it up - slightly.
```yaml
#--------------------------------------------

name: "ACTION1 - no nested actions"
inputs:
  # ... your inputs here

runs:
  using: "composite"
  steps:
    # ... your steps here

#--------------------------------------------
name: "ACTION2 - Using ACTION1"
inputs:
    # ... your inputs here
  nested:
    description: "Nested Action" # This is not necessary at the top level actions but I like to keep it consistent.
    required: true
    default: 'false'
  action_repository:
    description: "Internal use only - do not set"
    required: true
    default: "${{ github.action_repository }}"
  action_ref:
    description: "Internal use only - do not set"
    required: true
    default: "${{ github.action_ref }}"

runs:
  using: "composite"
  steps:
    - name: Checkout Action Repo - for Subsequent Actions
      uses: PythonGabe/SubsequentActionHelper@1.0.0
      with:
        repository: "${{ inputs.action_repository }}" # Or you can hard code the repository - DO not put "${{ github.action_repository }}" directly in the uses - it will then pull this project repository.
        action-ref: "${{ inputs.action_ref }}"
        action-repo-path: "./custom/actions" 
        private-repo-token: ${{ inputs.github_actions_token }}

    - name: Run Subsequent Action
      uses: "./custom/actions/ACTION1"
      with:
        # ... your inputs here
    
    # ... your steps here

#--------------------------------------------
name: "ACTION3 - Using ACTION2"
inputs:
  # ... your inputs here
  nested:
    description: "Nested Action" # This is not necessary at the top level actions but I like to keep it consistent.
    required: true
    default: 'false'
  action_repository:
    description: "Internal use only - do not set"
    required: true
    default: "${{ github.action_repository }}"
  action_ref:
    description: "Internal use only - do not set"
    required: true
    default: "${{ github.action_ref }}"

runs:
  using: "composite"
  steps:
    - name: Checkout Action Repo - for Subsequent Actions
      uses: PythonGabe/SubsequentActionHelper@1.0.0
      with:
        repository: "${{ inputs.action_repository }}" # Or you can hard code the repository - DO not put "${{ github.action_repository }}" directly in the uses - it will then pull this project repository.
        action-ref: "${{ inputs.action_ref }}"
        action-repo-path: "./custom/actions" 
        private-repo-token: ${{ inputs.github_actions_token }}

    - name: Run Subsequent Action
      uses: "./custom/actions/ACTION2"
      with:
        # ... your inputs here
        repository: "${{ inputs.action_repository }}" # Or you can hard code the repository - DO not put "${{ github.action_repository }}" directly in the uses - it will then pull this project repository.
        action-ref: "${{ inputs.action_ref }}"
        action-repo-path: "./custom/actions" 
        private-repo-token: ${{ inputs.github_actions_token }} # You can technically leave this blank as it won't be used in this case.

    # ... your steps here

#--------------------------------------------
```

## How it works
1. It will do a check to determine if the action is running in the same repository as your action or if it is being used in a different repository.
2. It will then put the repository in the 'action-repo-path' and then you can use the action from that path.
    - If it is being called in a different repository it will clone the repository to the 'action-repo-path'
    - If the action is in the same repository as the action itself and the action ref is the same as the action itself it will not clone the repository instead copy the contents of the repository to the 'action-repo-path'. ** If the action ref is different it will clone the repository to the 'action-repo-path' **
3. It adds a action-ref.txt with the ref of the action that is being used to the clone/copy to 'action-repo-path'
    - This avoids multiple clones if using the same ref 'action-repo-path' with the new ref. And also replace the contents of the 'action-repo-path' if the ref is different, allowing for the use of different versions of the action.

## Private Repository Tokens
If you are using a private repository you will need to provide a token to clone the repository. This token should be stored in the secrets of the repository and then passed in as a input to the action.
- For organizations - its best to create a organization secret and then use that secret in the action, so all your repositories can use the same token.
- The token will need access to the action repository.


## License
This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details