!#groovy

def splits = splitTests count(2)
def branches = [:]
for (int i = 0; i < splits.size(); i++) {
  def index = i // fresh variable per iteration; i will be mutated
  branches["split${i}"] = {
    node('master') {
      deleteDir()
      unstash 'sources'
      def exclusions = splits.get(index);
      writeFile file: 'exclusions.txt', text: exclusions.join("\n")
      sh "${tool 'M3'}/bin/mvn -B -Dmaven.test.failure.ignore test"
      junit 'target/surefire-reports/*.xml'
    }
  }
}
parallel branches
