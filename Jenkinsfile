stage 'Checkout'
// 'master' below indicates use master Jenkins only.  Can be replaced with the name of a node or a
// label used on several nodes.
node('master') {
  // Grabs an agent, checks out branch to test, and copies that branch.  This way all "sets" of test
  // are run on the same copy of the branch.  Pulling branch for testing and feeding it to each set
  // of tests instead of letting each set of tests pull it prevents follow-up commits from leading
  // different sets of test testing different code.
    git 'https://github.com/jenkinsci/parallel-test-executor-plugin-sample.git'
    stash name: 'sources', includes: 'pom.xml,src/'
}

stage 'Tests'
// Tests will be split into two sets of roughly equal runtime and run in parallel.
def splits = splitTests count(2)
def branches = [:]
for (int i = 0; i < splits.size(); i++) {
  def index = i // fresh variable per iteration; i will be mutated
  branches["split${i}"] = {
    node('master') {
      deleteDir()

      // Get copy of committed code is grabbed from archive/stash
      unstash 'sources'
      def exclusions = splits.get(index);
      writeFile file: 'exclusions.txt', text: exclusions.join("\n")

      // Failed tests are recorded and Jenkins continues with remaining tests.
      sh "${tool 'M3'}/bin/mvn -B -Dmaven.test.failure.ignore test"
      junit 'target/surefire-reports/*.xml'
    }
  }
}
parallel branches
