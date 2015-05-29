# Git shell command to output git log as json

    git log \
        --numstat \
        --format='id %h%nauthor %an%ndate %ai' $@ | \
        ruby -lawne '
            markers = %w{ id author date }
            if $F.empty?
              puts "\"changes\": ["
            else
              key = $F[0]
              if markers.include? key
                $F.shift
                value = $F.inject { |o, n| o + " " + n }
                puts key == "id" ? "]\}\},\{\"#{$F[0]}\":\{" : "\"#{key}\": \"#{value}\","
              else
                add = $F[0]
                del = $F[1]
                path = $F[2]
                puts "{\"additions\": #{add}, \"deletions\": #{del}, \"path\": \"#{path}\"},"
              end
            end
        ' | \
        ruby -wpe '
            BEGIN{puts "["}; END{puts "]\}\}]"}
        ' | \
        tr -d '\n' | \
        sed "s/,]/]/g; s/]}},//"

## JSON output:

    [
        {
            "481bccf": {
                "author": "Nathan Dao",
                "date": "2015-05-29 13:53:59 +0300",
                "changes": [
                    {
                        "additions": 2,
                        "deletions": 0,
                        "path": "path/to/Gemfile"
                    }
                ]
            }
        },
        {
             "8441ars": {
                "author": "Author 2",
                "date": "2015-05-29 12:33:59 +0300",
                "changes": [
                    {
                        "additions": 2,
                        "deletions": 0,
                        "path": "path/to/Gemfile"
                    },
                    {
                        "additions": 4,
                        "deletions": 0,
                        "path": "path/to/Gemfile.lock"
                    },
                    {
                        "additions": 8,
                        "deletions": 9,
                        "path": "path/to/app.rb"
                    },
                    {
                        "additions": 5,
                        "deletions": 2,
                        "path": "path/to/config.yml"
                    }
                ]
            }

        }
    ]
