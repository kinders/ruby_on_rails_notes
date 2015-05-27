Adding custom methods to Ability can allow for multiple conditions to be checked when defining abilities.

Define the ability.  This example defines each using a block, see [[Defining Abilities with Blocks]].  The conditions between '[ ]' are used to fetch records for use with accessible_by, see [[Fetching Records]].

```ruby
# In Ability
  when "voter"
    # Defined abilities as block
    can [:read], Ballot, ["organization_id IN (?)", user.organization_ids] do |ballot|
      is_ballot_readable?(user, ballot)
    end
    can [:vote], Ballot, ["organization_id IN (?)", user.organization_ids] do |ballot|
      is_ballot_voteable?(user, ballot)
    end
```

Define both methods referenced above.  These are defined after the initialize method.

```ruby
# In Ability
  def initialize(user)
    #Define abilities
  end

  def is_ballot_readable?(user, ballot)
    unless ballot.is_readable?
      raise CanCan::AccessDenied.new("Ballot cannot be accessed", [:read, :index, :show], Ballot)
    end

    unless user.organization_ids.include?(ballot.organization_id)
      raise CanCan::AccessDenied.new("Insufficient Authorization", [:read, :index, :show], Ballot)
    end
    
    return true
  end

  def is_ballot_voteable?(user, ballot)
    unless ballot.is_voteable?
      raise CanCan::AccessDenied.new("Ballot cannot be voted on at this time", [:vote], Ballot)
    end

    unless user.organization_ids.include?(ballot.organization_id)
      raise CanCan::AccessDenied.new("Insufficient Authorization", [:vote], Ballot)
    end
    
    return true
  end
```


Refer https://github.com/ryanb/cancan/issues/453