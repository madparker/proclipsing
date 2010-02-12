package processingplugin.core.createproject;

import java.io.ByteArrayInputStream;
import java.io.IOException;
import java.io.InputStream;
import java.lang.reflect.InvocationTargetException;
import java.net.URL;
import java.util.Vector;

import org.eclipse.core.runtime.CoreException;
import org.eclipse.core.runtime.IPath;
import org.eclipse.core.runtime.IProgressMonitor;
import org.eclipse.jdt.core.IAccessRule;
import org.eclipse.jdt.core.IClasspathAttribute;
import org.eclipse.jdt.core.IClasspathEntry;
import org.eclipse.jdt.core.JavaCore;
import org.eclipse.jdt.core.JavaModelException;
import org.eclipse.jdt.launching.JavaRuntime;
import org.eclipse.ui.actions.WorkspaceModifyOperation;
import org.eclipse.core.resources.IFile;
import org.eclipse.core.resources.IFolder;
import org.eclipse.core.resources.IProject;
import org.eclipse.core.resources.IProjectDescription;
import org.eclipse.core.resources.IWorkspaceRoot;
import org.eclipse.core.resources.ResourcesPlugin;

import processingplugin.core.Activator;
import processingplugin.util.Util;


/**
 * This job does the work of creating a Processing Project
 */
public class CreateProcessingProjectJob extends WorkspaceModifyOperation {

	private static final String BIN_DIR = "bin";
	private static final String SRC_DIR = "src";
	private static final String LIB_DIR = "lib";
	private static final String PROJECT_NAME_PLACEHOLDER = "%project_name%";
	private static final String PACKAGE_NAME_PLACEHOLDER = "%package_name%";
	
	private Vector<IClasspathEntry> classpath_entries;
	private String project_name;
	private String package_name;
	
	public CreateProcessingProjectJob(String projectName) {
		classpath_entries = new Vector<IClasspathEntry>();
		project_name = projectName;
	}
	
	@Override
	protected void execute(IProgressMonitor monitor) throws CoreException,
			InvocationTargetException, InterruptedException {
		
		IWorkspaceRoot wsroot = ResourcesPlugin.getWorkspace().getRoot();
		IProject project = wsroot.getProject(project_name);
		
		package_name = Util.projNametoPackage(project_name);
		
		project.create(monitor);
		project.open(monitor);
		
		addJavaNature(project, monitor);
		
		createBinFolder(project, monitor);
		createSrcFolder(project, monitor);
		createProjFolder(project, monitor);
		
		// create lib folder and set things in class path
		IFolder lib = createLibFolder(project, monitor);
		addProcessingToClasspath(lib, monitor);
		addSerialToClasspath(lib, monitor);
		
		addMyAppletSkelton(project, monitor);
		
		setProjectClassPath(project, monitor);
		
	}
	
	private void addMyAppletSkelton(IProject project, IProgressMonitor monitor) throws CoreException {
		IFolder srcDir = project.getFolder(SRC_DIR + "/" + package_name);
		IFile defaultApplet = srcDir.getFile(Util.strToCamelCase(project_name) + ".java");
		
		URL url = Activator.getDefault().getBundle().getResource(
				"template/PAppletTemplate.java");
		
		try{
			String templateStr = Util.convertStreamToString(url.openStream());

			String newApplet = Util.replaceAllSubString(templateStr, PACKAGE_NAME_PLACEHOLDER, package_name);
			newApplet = Util.replaceAllSubString(newApplet, PROJECT_NAME_PLACEHOLDER, Util.strToCamelCase(project_name));
			
			byte[] bArray = newApplet.getBytes();
			InputStream bais = new ByteArrayInputStream(bArray);
			
			defaultApplet.create(bais, true, monitor);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
	
	private void addProcessingToClasspath(
			IFolder lib, IProgressMonitor monitor) throws CoreException {
		URL url = Activator.getDefault().getBundle().getResource("processing/core.jar");
		IFile processingCore = lib.getFile("core.jar");
		try {
			processingCore.create(url.openStream(), true, monitor);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}
		
		classpath_entries.add(JavaCore.newLibraryEntry(processingCore.getFullPath(), null, null));
	}

	private void addSerialToClasspath(
			IFolder lib, IProgressMonitor monitor) throws CoreException {

		IClasspathAttribute[] attr = new IClasspathAttribute[]{
				JavaCore.newClasspathAttribute(JavaRuntime.CLASSPATH_ATTR_LIBRARY_PATH_ENTRY, "lib/")};
		
		URL serialJar1 = Activator.getDefault().getBundle().getResource("processing/libraries/serial/library/serial.jar");
		URL serialJar2 = Activator.getDefault().getBundle().getResource("processing/libraries/serial/library/RXTXcomm.jar");
		
		URL serialLib1 = Activator.getDefault().getBundle().getResource("processing/libraries/serial/library/rxtxSerial.dll");

		IFile processingCore1 = lib.getFile("serial.jar");
		IFile processingCore2 = lib.getFile("RXTXcomm.jar");
		
		try {
			processingCore1.create(serialJar1.openStream(), true, monitor);
			processingCore2.create(serialJar2.openStream(), true, monitor);
		} catch (IOException e) {
			// TODO Auto-generated catch block
			e.printStackTrace();
		}

		classpath_entries.add(JavaCore.newLibraryEntry(processingCore1.getFullPath(), 
				null, null, new IAccessRule [0], attr, false));
		classpath_entries.add(JavaCore.newLibraryEntry(processingCore2.getFullPath(), 
				null, null, new IAccessRule [0], attr, false));
	
	}
	
	
	private void addJavaNature(IProject project, IProgressMonitor monitor) throws CoreException {
		IProjectDescription desc = project.getDescription();
		desc.setNatureIds(new String[]{JavaCore.NATURE_ID});
		project.setDescription(desc, monitor);		
	}
	
	/**
	 * Adds the default Java classpath and then adds 
	 * everything in classpath_entries to the project classpath
	 * 
	 * @param proj
	 * @param monitor
	 * @throws JavaModelException
	 */
	private void setProjectClassPath(IProject proj, IProgressMonitor monitor) throws JavaModelException {
		IPath path  = JavaRuntime.newDefaultJREContainerPath();
		classpath_entries.add(JavaCore.newContainerEntry(path));
		
		// convert classpath_entries to IClasspathEntry[] and set classpath on java project
		JavaCore.create(proj).setRawClasspath(classpath_entries.toArray(
						new IClasspathEntry[classpath_entries.size()]), null);
	}
	
    private IFolder createSrcFolder(IProject project, IProgressMonitor monitor) throws CoreException {
        IFolder srcDir = project.getFolder(SRC_DIR);
        srcDir.create(true, true, monitor);
        classpath_entries.add(JavaCore.newSourceEntry(srcDir.getFullPath()));
        return srcDir;
    }
	
    private IFolder createProjFolder(IProject project, IProgressMonitor monitor) throws CoreException {
        IFolder projSrcDir = project.getFolder(SRC_DIR + "/" + package_name);
        projSrcDir.create(true, true, monitor);
        return projSrcDir;
    }
	
	private IFolder createLibFolder(IProject project, IProgressMonitor monitor) throws CoreException {
		IFolder libDir = project.getFolder(LIB_DIR);
		libDir.create(true, true, monitor);
		return libDir;
	}

	private IFolder createBinFolder(IProject project, IProgressMonitor monitor) throws CoreException {
		IFolder binDir = project.getFolder(BIN_DIR);
		binDir.create(true, true, monitor);
		JavaCore.create(project).setOutputLocation(binDir.getFullPath(), monitor);
		return binDir;
	}

}
