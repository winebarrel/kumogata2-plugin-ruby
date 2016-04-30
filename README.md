# kumogata2-plugin-ruby

It is the Ruby plug-in of [Kumogata2](https://github.com/winebarrel/kumogata2).

It convert the Ruby DSL to JSON.

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'kumogata2-plugin-ruby'
```

And then execute:

    $ bundle

Or install it yourself as:

    $ gem install kumogata2-plugin-ruby

## Example

```ruby
template do
  AWSTemplateFormatVersion "2010-09-09"

  Description (<<-EOS).undent
    Kumogata Sample Template
    You can use Here document!
  EOS

  Parameters do
    InstanceType do
      Default "t2.micro"
      Description "Instance Type"
      Type "String"
    end
  end

  Resources do
    myEC2Instance do
      Type "AWS::EC2::Instance"
      Properties do
        ImageId "ami-XXXXXXXX"
        InstanceType { Ref "InstanceType" }
        KeyName "your_key_name"

        UserData do
          Fn__Base64 (<<-EOS).undent
            #!/bin/bash
            yum install -y httpd
            service httpd start
          EOS
        end
      end
    end
  end
end
```

* `::` is converted to `__`
  * `Fn::GetAtt` => `Fn__GetAtt`
* `_{ ... }` is convered to Hash
  * `SecurityGroups [_{Ref "WebServerSecurityGroup"}]` => `{"SecurityGroups": [{"Ref": "WebServerSecurityGroup"}]}`
* `_path()` creates Hash that has a key of path
  * `_path("/etc/passwd-s3fs") { content "..." }` => `{"/etc/passwd-s3fs": {"content": "..."}}`


### Split a template file

* template.rb

```ruby
template do
  Resources do
    _include 'template2.rb', :ami_id => 'ami-XXXXXXXX'
  end
end
```

* template2.rb

```ruby
myEC2Instance do
  Type "AWS::EC2::Instance"
  Properties do
    ImageId args[:ami_id]
    InstanceType { Ref "InstanceType" }
    KeyName "your_key_name"
  end
end
```

* Converted JSON template

```javascript
{
  "Resources": {
    "myEC2Instance": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "ImageId": "ami-XXXXXXXX",
        "InstanceType": {
          "Ref": "InstanceType"
        },
        "KeyName": "your_key_name"
      }
    }
  }
}
```

### Post hook

You can run ruby script after building servers using `post()`.

```ruby
template do
  ...
end

post do |output|
  puts output
  #=> '{"WebsiteURL"=>"http://ec2-XX-XX-XX-XX.ap-northeast-1.compute.amazonaws.com"}'
end
```