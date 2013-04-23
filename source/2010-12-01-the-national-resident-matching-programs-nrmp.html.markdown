---
title: The National Resident Matching Program's (NRMP) Resident/Fellow Matching Algorithm, in Ruby
date: 2010-12-01 15:05
tags: programming, ruby
---

This implements the algorithm used by the National Resident Matching Program (NRMP) to match residency and fellowship applicants to programs.

[View this code as a gist.](https://gist.github.com/joeyAghion/719701)

The implementation:

    # Implements the algorithm used by the National Resident Matching Program
    # (NRMP) to match residency and fellowship applicants to programs. The
    # algorithm is described here:
    #   http://www.nrmp.org/fellow/algorithm.html
    # and again here:
    #   http://www.nrmp.org/res_match/about_res/algorithms.html
    #
    # The Test class computes the example match scenario described on those pages.

    module Match
  
      class Matcher
        attr_accessor :applicants, :programs

        def self.run(applicants_hash, programs_hash)
          applicants = applicants_hash.map { |name, list| Applicant.new(name, list) }
          programs = programs_hash.map { |name, attr| Program.new(name, attr[:positions], attr[:list]) }
          self.new(applicants, programs).run
        end
    
        def initialize(applicants, programs)
          self.applicants = applicants
          self.programs = {}
          programs.each do |program|
            self.programs[program.name] = program
          end
        end
    
        def run
          while remaining_applicants.any?
            puts "Round ##{round ||= 0; round += 1} (#{remaining_applicants.map(&:name).join(', ')})"
            remaining_applicants.each do |applicant|
              print "  #{applicant.name}: "
              while !applicant.exhausted_programs? && !applicant.matched?
                program = programs[applicant.next_program]
                print "#{program.name} "
                program.try_to_match(applicant)
              end
              puts ""
            end
          end
          self
        end
    
        def remaining_applicants
          applicants.reject{|applicant| applicant.matched? || applicant.exhausted_programs? }
        end
    
        def results_description
          "\nMatch results:\n".tap do |str|
            programs.each { |name, program| str << program.description + "\n" }
            str << "Unmatched: #{applicants.reject(&:matched?).map(&:name).join(', ')}"
          end
        end
      end
  
  
      class Program
        attr_accessor :name, :positions, :list, :matches
    
        def initialize(name, positions, list)
          self.name = name
          self.positions = positions
          self.list = list
          self.matches = []
        end
    
        def try_to_match(applicant)
          if list.index(applicant.name)
            print "(#{list.index(applicant.name)}) "
            matches << applicant
            applicant.matched = true
            cull_matches
          else
            print "(unranked) "
          end
        end

        def cull_matches
          while matches.size > positions do
            reject = matches.sort_by{|applicant| list.index(applicant.name) }.last
            reject.matched = false
            matches.delete(reject)
            print "[culled #{reject.name}] "
          end
        end
    
        def description
          "#{name}: #{matches.map(&:name).join(', ')} #{"(#{positions - matches.size} unfilled)" if matches.size < positions}"
        end
      end
  
  
      class Applicant
        attr_accessor :name, :list, :matched
    
        def initialize(name, list)
          self.name = name
          self.list = list
          @ranking_position = -1
        end
    
        def exhausted_programs?
          @ranking_position >= self.list.size - 1
        end
    
        def next_program
          raise "No ranked programs available!" if exhausted_programs?
          @ranking_position += 1
          self.list[@ranking_position]
        end
    
        def matched?
          matched
        end
      end
    end

    class Test

      PROGRAMS = {
        'city' => {:positions => 2, :list => ['garcia', 'hassan', 'eastman', 'anderson', 'brown', 'chen', 'davis', 'ford']},
        'general' => {:positions => 2, :list => ['brown', 'eastman', 'hassan', 'anderson', 'chen', 'davis', 'garcia']},
        'mercy' => {:positions => 2, :list => ['chen', 'garcia']},
        'state' => {:positions => 2, :list => ['brown', 'eastman', 'anderson', 'chen', 'hassan', 'ford', 'davis', 'garcia']}
      }

      APPLICANTS = {
        'anderson' => ['city'],
        'brown' => ['city', 'mercy'],
        'chen' => ['city', 'mercy'],
        'davis' => ['mercy', 'city', 'general', 'state'],
        'eastman' => ['city', 'mercy', 'state', 'general'],
        'ford' => ['city', 'general', 'mercy', 'state'],
        'garcia' => ['city', 'mercy', 'state', 'general'],
        'hassan' => ['state', 'city', 'mercy', 'general']
      }

      def self.main
        matcher = Match::Matcher.run(APPLICANTS, PROGRAMS)
        puts matcher.results_description
      end
    end

    Test.main

And, the results:

    Round #1 (chen, anderson, brown, garcia, davis, ford, eastman, hassan)
      chen: city (5) 
      anderson: city (3) 
      brown: city (4) [culled chen] 
      garcia: city (0) [culled brown] 
      davis: mercy (unranked) city (6) [culled davis] general (5) 
      ford: city (7) [culled ford] general (unranked) mercy (unranked) state (5) 
      eastman: city (2) [culled anderson] 
      hassan: state (4) 
    Round #2 (chen, brown)
      chen: mercy (0) 
      brown: mercy (unranked) 

    Match results:
    city: garcia, eastman 
    mercy: chen (1 unfilled)
    general: davis (1 unfilled)
    state: ford, hassan 
    Unmatched: anderson, brown
