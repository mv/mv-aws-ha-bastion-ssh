# vim:ft=ruby
#
# To remember: Ruby/Rake shell commands
#    Ruby
#       %x{ cmd } or `cmd`  - executes 'cmd' and gives cmd OUTPUT
#       system 'cmd'        - executes 'cmd' and gives OS level return code.
#       exec 'cmd'          - executes 'cmd' in the current process and finalizes it.
#    Rake
#       sh 'cmd'            - prints 'cmd' before executing it and gives cmd OUTPUT#       sh 'cmd'            - prints 'cmd' before executing it and gives cmd OUTPUT
#       sh 'cmd' do |return,output| - executes 'cmd' and gets its OUTPUT and OS return code.
#
# To remeber (2): string quotes
#   '', %q{}, %q[], %q//, %q|| - single quotes
#   "", %Q{}, %Q[], %Q//, %Q|| - double quotes
#   "", %{} , %[] , %// , %||  - double quotes
#
# To remeber (3): heredocs
#   # heredoc, idented, interpolation
#   sh <<-CMD
#       cmd
#       cmd #{var}
#   CMD
#
#   # heredoc, idented, interpolation, remove identation
#   sh <<-'CMD.gsub(/^[ ]*/,'') -
#       cmd
#       cmd #{var}
#   CMD
#
#   # heredoc, idented, no interpolation (as single quotes)
#   sh <<-'CMD'
#       cmd
#       cmd #{var}
#   CMD
#

require 'json'

###
### Yeah, I know: shameful use of 'exec'...
###

desc "default: list rake tasks"
task :default do
  exec "rake --tasks"
end

###
### Settings
###
@config   = ENV['config'] || "config/example.yml"
@template = "mv-aws-ha-bastion-ssh.template.json"


###
### Tasks
###
desc "cloudformation: validate-template"
task :'cf-validate' do
  sh %Q{
    aws cloudformation validate-template --output table \
        --template-body file://#{@template}
  }
end

desc "cloudformation: create-stack"
task :'cf-create' do
  sh %Q{
    aws cloudformation create-stack \
        --template-body file://#{@template} \
        --stack-name #{@stack}      \
        --parameters \
          ParameterKey=KeyName,ParameterValue=internal-mv.dev.id_rsa \
        --output json
  }
end



###
### AMI
###
desc "most recent Amazon Linux AMIs"
task :"latest-amis" do
  sh %Q{
    aws ec2 describe-images \
      --owners amazon                           \
      --filters                                 \
          "Name=virtualization-type,Values=hvm" \
          "Name=architecture,Values=x86_64"     \
          "Name=root-device-type,Values=ebs"    \
      --query 'Images[?Platform != `windows`].[ImageId,Name]'  \
      --output text | sort -k 2 | grep 'amzn-ami'              \
  }
end

desc "list chosen AMI in all regions: rake list-ami-in-regions latest='ami-description'"
task :"list-ami-in-regions" do

  latest_ami = "amzn-ami-minimal-hvm-2016.03.1.x86_64-ebs"
  latest     = ENV['latest'] || latest_ami

  mapping = {}
  regions = %x{ aws ec2 describe-regions --output json --query 'Regions[].[RegionName]' --output text }

  regions.split.sort.each do |region|
    res = %x{
      aws ec2 describe-images \
        --region #{region}    \
        --filters "Name=name,Values=#{latest}" \
        --query 'Images[].[ImageId,CreationDate,Name]' \
        --output text
    }
    # display result
    printf "%-20s %s", region, res

    # build 'mapping' hash
    mapping[region] = { "AMI" => res.split[0] }
  end

  printf "Latest AMI mappings:\n"
  puts JSON.pretty_generate(mapping)

end

