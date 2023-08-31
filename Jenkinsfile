import groovy.json.JsonSlurper
import groovy.json.*
import java.io.OutputStreamWriter
import java.lang.String
import groovy.transform.Field
import jenkins.model.*
import java.io.FileReader;
import java.util.Iterator;
import java.util.Map
import groovy.io.FileType
import groovy.json.JsonSlurperClassic
import hudson.model.*

def NAME_ZIPFILE
def ZIP_WORKFLOW
def jsonResponse
String fileuploadUrl
String postuploadfileuploadUrl
def list = []
String taskID
String path_zipfile
String zip_workflowfilepath
String includefilenames
String filedata
def path_filestobedeployed 
def postdeploymentfilepath
def statuscode
def objectstatus
def objectsecondstatus
String statusDetail1msg = ""
def status_final
def statusvalue
String includedfile = ""
def tenant="DS"
def tenantstobeexcluded

def postdeployment_zipfile
def path_postdeploymentfiles
def path_postdeployment_zipfile
String postdeploymentfile = ""
def dopostdeployment = true
def parameters



@NonCPS
def makeApiCallAndGetResponse(String taskID) {
    def post = new URL(parameters[0].tenanturl+"/api/requesttrackingservice/get").openConnection() as HttpURLConnection
    def requestData = '{"params":{"query":{"id":"' + taskID + '","filters":{"typesCriterion":["tasksummaryobject"]}},"fields":{"attributes":["_ALL"],"relationships":["_ALL"]},"options":{"maxRecords":1000}}}'
    def message = '{"message":"this is a message"}'

    post.setRequestMethod("POST")
    post.setDoOutput(true)
    post.setRequestProperty("Content-Type", "application/zip")
    post.setRequestProperty("x-rdp-version", "8.1")
    post.setRequestProperty("x-rdp-tenantId", parameters[0].xrdptenantid)
    post.setRequestProperty("x-rdp-clientId", "rdpclient")
    post.setRequestProperty("x-rdp-userId", parameters[0].xrdpuserid)
    post.setRequestProperty("x-rdp-userRoles", parameters[0].xrdpuserrole)
    post.setRequestProperty("auth-client-id", parameters[0].authclientid)
    post.setRequestProperty("auth-client-secret", parameters[0].authclientsecret)

    post.connect()

    // Write the request data to the output stream
    def outputStream = post.getOutputStream() as OutputStream
    outputStream.write(requestData.getBytes("UTF-8"))
    outputStream.flush()
    outputStream.close()

    // Get the response from the input stream
    def responseCode = post.getResponseCode()
    def inputStream = (responseCode == HttpURLConnection.HTTP_OK) ? post.getInputStream() : post.getErrorStream()
    def reader = new BufferedReader(new InputStreamReader(inputStream))
    def responseBuilder = new StringBuilder()
    String line
    while ((line = reader.readLine()) != null) {
        responseBuilder.append(line)
    }
    def responsess = responseBuilder.toString()

    reader.close()
    inputStream.close()
    post.disconnect()

    return responsess
}

pipeline {
    agent any

    stages {        
        stage('Path Variables Initialization') {
            steps {
                script {
                    NAME_ZIPFILE = "${env.BRANCH_NAME}.zip"
                    postdeployment_zipfile = "postdeployment.zip"
                    path_filestobedeployed = "${env.WORKSPACE}/deployment-artifacts/filestobedeployed.json"
                    path_postdeploymentfiles = "${env.WORKSPACE}/deployment-artifacts/postdeploymentconfig.json"
                    path_zipfile = "${env.WORKSPACE}/${NAME_ZIPFILE}"
                    path_postdeployment_zipfile = "${env.WORKSPACE}/${postdeployment_zipfile}"
                    def paramsJson = readFile(file: "/deployment-artifacts/parameters.json")
                    parameters = readJSON(text: paramsJson)

                    println("NAME_ZIPFILE: " + NAME_ZIPFILE)
                    println("ZIP_WORKFLOW: " + ZIP_WORKFLOW)
                    println("path_filestobedeployed: " + path_filestobedeployed)
                    println("path_zipfile: " + path_zipfile)
                }
            }
        }

        stage('Zipping Files') {
            steps {
                script {
                    def includedFileObject = new File(path_filestobedeployed)
                    def includedFileJSONObject = new JsonSlurperClassic().parse(includedFileObject)

                    def includedFilenamesString = includedFileJSONObject.filename

                    println("includedFilenamesString: " + includedFilenamesString)

                    if (includedFilenamesString.size() > 1) {
                        // The string contains a comma
                        for (int i = 0; i < includedFilenamesString.size(); i++) {
                            if (i >= 1) {
                                includedfile += ","
                            }
                            includedFilenamesString[i].trim().replaceAll("\\[|\\]", "")
                            includedfile += "'${env.WORKSPACE}" + "\\" + includedFilenamesString[i] + "'"
                        }
                        println("Final included files: " + includedfile)
                    } else {
                        // The string does not contain a comma
                        // Assuming includedFilenamesString is an ArrayList, extract a string element
                        String filenamesString = includedFilenamesString.get(0)

                        // Trim leading and trailing whitespace, and remove square brackets
                        filenamesString = filenamesString.trim().replaceAll("\\[|\\]", "")
                        includedfile = "'${env.WORKSPACE}" + "\\" + filenamesString + "'"
                        println("No comma present" + includedfile)
                    }

                    println("Final included files: " + includedfile)
                    println("Filenames processed successfully... Moving to zipping..")

                    bat """
                    powershell.exe -Command "if (Test-Path '${path_zipfile}') { Remove-Item '${path_zipfile}' }"
                    powershell.exe -Command "Compress-Archive -Path @(${includedfile}) -DestinationPath '${path_zipfile}'"
                    """
                    if (!fileExists(path_zipfile)) {
                        error("Failed to create the zip file.")
                    } else {
                        println("Files zipped successfully...")
                    }
                }
            }
        }
        stage('Prepare Upload') {
            steps {
                script {
                    def jsonobject = "{\"binaryStreamObject\":{\"id\":\"guid\",\"type\":\"seedDataStream\",\"properties\":{\"objectKey\":\"${NAME_ZIPFILE}\",\"originalFileName\":\"${NAME_ZIPFILE}\"}}}"

                    def post = new URL(parameters[0].tenanturl+"/api/binarystreamobjectservice/prepareUpload").openConnection() 
                    def message = '{"message":"this is a message"}'
                    post.setRequestMethod("POST")
                    post.setDoOutput(true)
                    post.setRequestProperty("Content-Type", "application/json")
                    post.setRequestProperty("x-rdp-version", "8.1")
                    post.setRequestProperty("x-rdp-tenantId", parameters[0].xrdptenantid)
                    post.setRequestProperty("x-rdp-clientId", "rdpclient")
                    post.setRequestProperty("x-rdp-userId", parameters[0].xrdpuserid)
                    post.setRequestProperty("x-rdp-userRoles", parameters[0].xrdpuserrole)
                    post.setRequestProperty("auth-client-id", parameters[0].authclientid)
                    post.setRequestProperty("auth-client-secret", parameters[0].authclientsecret)
                    post.connect()

                    OutputStreamWriter out = new OutputStreamWriter(post.getOutputStream())
                    out.write(jsonobject)
                    out.close()
                    def statuscode1 = post.getResponseCode()
                    String outputObj = post.getInputStream().getText()
                    println("StatusCode=" + statuscode1)
                    println("outputObj=" + outputObj)
                    Map jsonContent = (Map) new JsonSlurper().parseText(outputObj)
                    String data1 = jsonContent.response.binaryStreamObjects.data
                    String[] arrOfStr = data1.split(",")
                    
                    for (int i = 0; i < arrOfStr.length; i++) {
                        String[] arrOfurl = arrOfStr[i].split("uploadURL=")
                        
                        for (int p = 1; p < arrOfurl.length; p++) {
                            fileuploadUrl = arrOfurl[p] - "}}]"
                            println("data[" + p + "]: " + arrOfurl[p])
                        }
                    }
                    
                    println(fileuploadUrl)
                }
            }
        }

        stage('Deploy Files') {
            steps {
                script {
                    echo "==== Deploying folder ===="

                    def encodedFileuploadUrl = fileuploadUrl.replaceAll('%', '%%')

                    bat """
                        curl -v -X PUT "${encodedFileuploadUrl}" ^
                        --header "x-ms-meta-x_rdp_userroles: ${parameters[0].xrdpuserrole}" ^
                        --header "x-ms-meta-x_rdp_tenantid: ${parameters[0].xrdptenantid}" ^
                        --header "x-ms-meta-originalfilename: ${NAME_ZIPFILE}" ^
                        --header "x-ms-blob-content-disposition: attachment; filename=${NAME_ZIPFILE}" ^
                        --header "x-ms-meta-type: disposition" ^
                        --header "x-ms-meta-x_rdp_clientid: rdpclient" ^
                        --header "x-ms-meta-x_rdp_userid: ${parameters[0].xrdpuserid}" ^
                        --header "x-ms-meta-binarystreamobjectid: guid" ^
                        --header "x-ms-blob-type: BlockBlob" ^
                        --header "Content-Type: application/zip" ^
                        --data-binary "@${path_zipfile}"
                    """
                }
            }
        }

        stage('Upload to Tenant'){
                steps{
                    script{
                        echo "====Deployment====="
                        def jsonobject = "{\"adminObject\":{\"id\":\"someguid\",\"type\":\"adminObject\",\"properties\":{\"flushConfig\":false,\"storageType\":\"stream\",\"objectKey\":\"${NAME_ZIPFILE}\",\"tenantId\":\"${parameters[0].xrdptenantid}\",\"retryCount\":1,\"sleepTime\":1000}}}"
                        def post = new URL(parameters[0].tenanturl+"/api/adminservice/deploytenantseed").openConnection();
                        def message = '{"message":"this is a message"}'
                        post.setRequestMethod("POST")
                        post.setDoOutput(true)
                        post.setRequestProperty("Content-Type","application/zip")
                        post.setRequestProperty("x-rdp-version","8.1")
                        
                        post.setRequestProperty("x-rdp-clientId","rdpclient")
                        
                        post.setRequestProperty("x-rdp-userId",parameters[0].xrdpuserid)
                        
                        post.setRequestProperty("x-rdp-userRoles",parameters[0].xrdpuserrole)
                        
                        post.setRequestProperty("auth-client-id",parameters[0].authclientid)
                        
                        post.setRequestProperty("auth-client-secret",parameters[0].authclientsecret)
                        OutputStreamWriter out = new OutputStreamWriter(post.getOutputStream());
                        out.write(jsonobject);
                        out.close();
                        post.getOutputStream().write(message.getBytes("UTF-8"));
                        def statuscode2 = post.getResponseCode();
                        def outputObj = post.getInputStream().getText();
                        println("statusCode=" +statuscode2)
                        Map jsonContent = (Map) new JsonSlurper().parseText(outputObj)
                        println(jsonContent)
                        def status = jsonContent.response.status
                        def totalRecords= jsonContent.response.totalRecords
                        taskID = jsonContent.response.statusDetail.taskId
                        println("Status="+status);
                        println("Taskid="+taskID)
                        println("TotalRecod="+totalRecords);

                    }    
            }
        }
        
        stage('Verify Upload') {
            steps {
                script {
                    def taskstatus = false
                    def responsess

                    while (!taskstatus) {
                        responsess = makeApiCallAndGetResponse(taskID)

                        // Process the response
                        println("task_mssage response: " + responsess)

                        node {
                            // Run non-serializable operations on the agent
                            def jsonSlurper = new groovy.json.JsonSlurper()
                            def jsonContent = jsonSlurper.parseText(responsess.trim())
                            def totalRecord = jsonContent.response.totalRecords

                            if (totalRecord == 1) {
                                objectstatus = jsonContent.response.requestObjects[0].data.attributes.status.values[0].value
                                //println("=========== objecttttt=found====" + objectstatus)
                                if (objectstatus == "Completed" || objectstatus == "Completed With Errors" || objectstatus == "Errored") {
                                    taskstatus = true
                                    println("Task is completed with status: "+objectstatus+" \nMoving to Post-Deployment.")
                                }
                            } else {
                                //statusDetail1msg = jsonContent.response.statusDetail.messages[0].message
                                println("===========no objecttttt found. exiting current stage.=====" + statusDetail1msg)
                                taskstatus=true
                            }
                        }

                        if (!taskstatus) {
                            println("Task is in progress. Re-checking in 30 seconds.")
                            sleep(30)
                        }
                    }
                }
            }
        }

        stage('Post Deployment Stage'){
            steps{
                script {
                    println("Checking for files for Post Deployment")

                    def postdeploymentfilesObject = new File(path_postdeploymentfiles)
                    def postdeploymentFileJSONObject = new JsonSlurperClassic().parse(postdeploymentfilesObject)

                    def postdeploymentFilenamesString = postdeploymentFileJSONObject.filename

                    println("postdeploymentFilenamesString: " + postdeploymentFilenamesString)

                    if(postdeploymentFilenamesString.size()>0){
                        if (postdeploymentFilenamesString.size() > 1) {
                            // The string contains a comma
                            for (int i = 0; i < postdeploymentFilenamesString.size(); i++) {
                                if (i >= 1) {
                                    postdeploymentfile += ","
                                }
                                postdeploymentFilenamesString[i].trim().replaceAll("\\[|\\]", "")
                                postdeploymentfile += "'${env.WORKSPACE}" + "\\" + postdeploymentFilenamesString[i] + "'"
                            }
                            println("Final post deployment files: " + postdeploymentfile)
                        } else {
                            // The string does not contain a comma
                            // Assuming postdeploymentFilenamesString is an ArrayList, extract a string element
                            String filenamesString = postdeploymentFilenamesString.get(0)

                            // Trim leading and trailing whitespace, and remove square brackets
                            filenamesString = filenamesString.trim().replaceAll("\\[|\\]", "")
                            postdeploymentfile = "'${env.WORKSPACE}" + "\\" + filenamesString + "'"
                            println("No comma present" + postdeploymentfile)
                        }
                    }
                    else{
                        println("No files mentioned in postdeploymentconfig.json. Skipping post-deployment.")
                        dopostdeployment=false
                    }

                    println("Final included files: " + postdeploymentfile)
                    println("Filenames processed successfully... Moving to zipping..")

                    bat """
                    powershell.exe -Command "if (Test-Path '${path_postdeployment_zipfile}') { Remove-Item '${path_postdeployment_zipfile}' }"
                    powershell.exe -Command "Compress-Archive -Path @(${postdeploymentfile}) -DestinationPath '${path_postdeployment_zipfile}'"
                    """
                    if (!fileExists(path_postdeployment_zipfile)) {
                        error("Failed to create the zip file.")
                    } else {
                        println("Files zipped successfully...")
                    }
                } 
                script {
                    def jsonobject = "{\"binaryStreamObject\":{\"id\":\"guid\",\"type\":\"seedDataStream\",\"properties\":{\"objectKey\":\"${postdeployment_zipfile}\",\"originalFileName\":\"${postdeployment_zipfile}\"}}}"

                    def post = new URL(parameters[0].tenanturl+"/api/binarystreamobjectservice/prepareUpload").openConnection() 
                    def message = '{"message":"this is a message"}'
                    post.setRequestMethod("POST")
                    post.setDoOutput(true)
                    post.setRequestProperty("Content-Type", "application/json")
                    post.setRequestProperty("x-rdp-version", "8.1")
                    post.setRequestProperty("x-rdp-tenantId", parameters[0].xrdptenantid)
                    post.setRequestProperty("x-rdp-clientId", "rdpclient")
                    post.setRequestProperty("x-rdp-userId", parameters[0].xrdpuserid)
                    post.setRequestProperty("x-rdp-userRoles", parameters[0].xrdpuserrole)
                    post.setRequestProperty("auth-client-id", parameters[0].authclientid)
                    post.setRequestProperty("auth-client-secret", parameters[0].authclientsecret)
                    post.connect()

                    OutputStreamWriter out = new OutputStreamWriter(post.getOutputStream())
                    out.write(jsonobject)
                    out.close()
                    def statuscode1 = post.getResponseCode()
                    String outputObj = post.getInputStream().getText()
                    println("StatusCode=" + statuscode1)
                    println("outputObj=" + outputObj)
                    Map jsonContent = (Map) new JsonSlurper().parseText(outputObj)
                    String data1 = jsonContent.response.binaryStreamObjects.data
                    String[] arrOfStr = data1.split(",")
                    
                    for (int i = 0; i < arrOfStr.length; i++) {
                        String[] arrOfurl = arrOfStr[i].split("uploadURL=")
                        
                        for (int p = 1; p < arrOfurl.length; p++) {
                            fileuploadUrl = arrOfurl[p] - "}}]"
                            println("data[" + p + "]: " + arrOfurl[p])
                        }
                    }
                    
                    println(fileuploadUrl)
                }  
                script {
                    echo "==== Deploying folder ===="

                    def encodedFileuploadUrl = fileuploadUrl.replaceAll('%', '%%')

                    bat """
                        curl -v -X PUT "${encodedFileuploadUrl}" ^
                        --header "x-ms-meta-x_rdp_userroles: ${parameters[0].xrdpuserrole}" ^
                        --header "x-ms-meta-x_rdp_tenantid: ${parameters[0].xrdptenantid}" ^
                        --header "x-ms-meta-originalfilename: ${NAME_ZIPFILE}" ^
                        --header "x-ms-blob-content-disposition: attachment; filename=${NAME_ZIPFILE}" ^
                        --header "x-ms-meta-type: disposition" ^
                        --header "x-ms-meta-x_rdp_clientid: rdpclient" ^
                        --header "x-ms-meta-x_rdp_userid: ${parameters[0].xrdpuserid}" ^
                        --header "x-ms-meta-binarystreamobjectid: guid" ^
                        --header "x-ms-blob-type: BlockBlob" ^
                        --header "Content-Type: application/zip" ^
                        --data-binary "@${path_zipfile}"
                    """
                } 
                script{
                    echo "====Deployment====="
                    def jsonobject = "{\"adminObject\":{\"id\":\"someguid\",\"type\":\"adminObject\",\"properties\":{\"flushConfig\":false,\"storageType\":\"stream\",\"objectKey\":\"${postdeployment_zipfile}\",\"tenantId\":\"${parameters[0].xrdptenantid}\",\"retryCount\":1,\"sleepTime\":1000}}}"
                    def post = new URL(parameters[0].tenanturl+"/api/adminservice/deploytenantseed").openConnection();
                    def message = '{"message":"this is a message"}'
                    post.setRequestMethod("POST")
                    post.setDoOutput(true)
                    post.setRequestProperty("Content-Type","application/zip")
                    post.setRequestProperty("x-rdp-version","8.1")
                    
                    post.setRequestProperty("x-rdp-clientId","rdpclient")
                        
                    post.setRequestProperty("x-rdp-userId",parameters[0].xrdpuserid)
                    
                    post.setRequestProperty("x-rdp-userRoles",parameters[0].xrdpuserrole)
                    
                    post.setRequestProperty("auth-client-id",parameters[0].authclientid)
                    
                    post.setRequestProperty("auth-client-secret",parameters[0].authclientsecret)
                    OutputStreamWriter out = new OutputStreamWriter(post.getOutputStream());
                    out.write(jsonobject);
                    out.close();
                    post.getOutputStream().write(message.getBytes("UTF-8"));
                    def statuscode2 = post.getResponseCode();
                    def outputObj = post.getInputStream().getText();
                    println("statusCode=" +statuscode2)
                    Map jsonContent = (Map) new JsonSlurper().parseText(outputObj)
                    println(jsonContent)
                    def status = jsonContent.response.status
                    def totalRecords= jsonContent.response.totalRecords
                    taskID = jsonContent.response.statusDetail.taskId
                    println("Status="+status);
                    println("Taskid="+taskID)
                    println("TotalRecod="+totalRecords);

                }  
                script {
                    def taskstatus = false
                    def responsess

                    while (!taskstatus) {
                        responsess = makeApiCallAndGetResponse(taskID)

                        // Process the response
                        println("task_mssage response: " + responsess)

                        node {
                            // Run non-serializable operations on the agent
                            def jsonSlurper = new groovy.json.JsonSlurper()
                            def jsonContent = jsonSlurper.parseText(responsess.trim())
                            def totalRecord = jsonContent.response.totalRecords

                            if (totalRecord == 1) {
                                objectstatus = jsonContent.response.requestObjects[0].data.attributes.status.values[0].value
                                //println("=========== objecttttt=found====" + objectstatus)
                                if (objectstatus == "Completed" || objectstatus == "Completed With Errors" || objectstatus == "Errored") {
                                    taskstatus = true
                                    println("Task is completed with status: "+objectstatus+" \nMoving to Post-Deployment.")
                                }
                            } else {
                                //statusDetail1msg = jsonContent.response.statusDetail.messages[0].message
                                println("===========no objecttttt found. exiting current stage.=====" + statusDetail1msg)
                                taskstatus=true
                            }
                        }

                        if (!taskstatus) {
                            println("Task is in progress. Re-checking in 30 seconds.")
                            sleep(30)
                        }
                    }
                }              
            }
        }

    }
}
