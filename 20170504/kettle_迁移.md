[TOC]

# kettle 迁移



kettle windows linux removal

由于“想哭”病毒导致windows无法正常作业，为了后面考虑，想将etl服务器放到linux上，一些迁移过程终遇到的问题，记录在文件，以备后续使用

备份数据库脚本

exp/imp

本次的kettle的资源库是Oracle，故而整个备份这个用户就可以

修改相关参数

1、ftp的下载、上传的路径

2、文件生成的路径

3、各个数据库的相关信息

由于本次是自己独立自主完成的数据整合流程，方案名等起的不是很规范，这次刚好将命名规范

========================================================================================

本次遇到报错信息如下：

1-1.报表不存在

2017/05/18 06:16:34 - SPOON4 - at org.pentaho.di.core.database.Database.openQuery(Database.java:1776)

2017/05/18 06:16:34 - SPOON4 - ... 39 more

2017/05/18 06:16:35 - SPOON4 - ERROR (version 5.2.0.0, build 1 from 2014-09-30_19-48-28 by buildguy) : 一个数据库错误发生在从资源库文件读取转换时

2017/05/18 06:16:35 - SPOON4 - org.pentaho.di.core.exception.KettleDatabaseException:

2017/05/18 06:16:35 - SPOON4 - ERROR executing query

2017/05/18 06:16:35 - SPOON4 - ORA-00942: 表或视图不存在

/

2017/05/18 06:17:14 - SPOON4 - ERROR (version 5.2.0.0, build 1 from 2014-09-30_19-48-28 by buildguy) : 当读共享文件时发生错误(继续加载): org.pentaho.di.core.exception.KettleException:

2017/05/18 06:17:14 - SPOON4 - 不能从资源库里读取分区模式

2017/05/18 06:17:14 - SPOON4 -

2017/05/18 06:17:14 - SPOON4 - ERROR executing query

2017/05/18 06:17:14 - SPOON4 - ORA-00942: 表或视图不存在

2017/05/18 06:17:14 - SPOON4 - ERROR (version 5.2.0.0, build 1 from 2014-09-30_19-48-28 by buildguy) : org.pentaho.di.core.exception.KettleException:

2017/05/18 06:17:14 - SPOON4 - 不能从资源库里读取分区模式

2017/05/18 06:17:14 - SPOON4 -

2017/05/18 06:17:14 - SPOON4 - ERROR executing query

2017/05/18 06:17:14 - SPOON4 - ORA-00942: 表或视图不存在

2017/05/18 06:17:14 - SPOON4 -

2017/05/18 06:17:14 - SPOON4 -

2017/05/18 06:17:14 - SPOON4 -

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.repository.kdr.delegates.KettleDatabaseRepositoryTransDelegate.readPartitionSchemas(KettleDatabaseRepositoryTransDelegate.java:1036)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.repository.kdr.delegates.KettleDatabaseRepositoryTransDelegate.readTransSharedObjects(KettleDatabaseRepositoryTransDelegate.java:1493)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.repository.kdr.delegates.KettleDatabaseRepositoryTransDelegate.loadTransformation(KettleDatabaseRepositoryTransDelegate.java:499)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.repository.kdr.KettleDatabaseRepository.loadTransformation(KettleDatabaseRepository.java:278)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.ui.trans.dialog.TransLoadProgressDialog$1.run(TransLoadProgressDialog.java:92)

2017/05/18 06:17:14 - SPOON4 - at org.eclipse.jface.operation.ModalContext.runInCurrentThread(ModalContext.java:369)

2017/05/18 06:17:14 - SPOON4 - at org.eclipse.jface.operation.ModalContext.run(ModalContext.java:313)

2017/05/18 06:17:14 - SPOON4 - at org.eclipse.jface.dialogs.ProgressMonitorDialog.run(ProgressMonitorDialog.java:495)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.ui.trans.dialog.TransLoadProgressDialog.open(TransLoadProgressDialog.java:105)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.ui.spoon.Spoon.openFile(Spoon.java:4238)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.ui.spoon.Spoon.openFile(Spoon.java:4160)

2017/05/18 06:17:14 - SPOON4 - at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)

2017/05/18 06:17:14 - SPOON4 - at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)

2017/05/18 06:17:14 - SPOON4 - at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)

2017/05/18 06:17:14 - SPOON4 - at java.lang.reflect.Method.invoke(Method.java:606)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.ui.xul.impl.AbstractXulDomContainer.invoke(AbstractXulDomContainer.java:313)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.ui.xul.impl.AbstractXulComponent.invoke(AbstractXulComponent.java:157)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.ui.xul.impl.AbstractXulComponent.invoke(AbstractXulComponent.java:141)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.ui.xul.jface.tags.JfaceMenuitem.access$100(JfaceMenuitem.java:43)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.ui.xul.jface.tags.JfaceMenuitem$1.run(JfaceMenuitem.java:106)

2017/05/18 06:17:14 - SPOON4 - at org.eclipse.jface.action.Action.runWithEvent(Action.java:498)

2017/05/18 06:17:14 - SPOON4 - at org.eclipse.jface.action.ActionContributionItem.handleWidgetSelection(ActionContributionItem.java:545)

2017/05/18 06:17:14 - SPOON4 - at org.eclipse.jface.action.ActionContributionItem.access$2(ActionContributionItem.java:490)

2017/05/18 06:17:14 - SPOON4 - at org.eclipse.jface.action.ActionContributionItem$5.handleEvent(ActionContributionItem.java:402)

2017/05/18 06:17:14 - SPOON4 - at org.eclipse.swt.widgets.EventTable.sendEvent(Unknown Source)

2017/05/18 06:17:14 - SPOON4 - at org.eclipse.swt.widgets.Widget.sendEvent(Unknown Source)

2017/05/18 06:17:14 - SPOON4 - at org.eclipse.swt.widgets.Display.runDeferredEvents(Unknown Source)

2017/05/18 06:17:14 - SPOON4 - at org.eclipse.swt.widgets.Display.readAndDispatch(Unknown Source)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.ui.spoon.Spoon.readAndDispatch(Spoon.java:1310)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.ui.spoon.Spoon.waitForDispose(Spoon.java:7931)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.ui.spoon.Spoon.start(Spoon.java:9202)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.ui.spoon.Spoon.main(Spoon.java:648)

2017/05/18 06:17:14 - SPOON4 - at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)

2017/05/18 06:17:14 - SPOON4 - at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)

2017/05/18 06:17:14 - SPOON4 - at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)

2017/05/18 06:17:14 - SPOON4 - at java.lang.reflect.Method.invoke(Method.java:606)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.commons.launcher.Launcher.main(Launcher.java:92)

2017/05/18 06:17:14 - SPOON4 - Caused by: org.pentaho.di.core.exception.KettleDatabaseException:

2017/05/18 06:17:14 - SPOON4 - ERROR executing query

2017/05/18 06:17:14 - SPOON4 - ORA-00942: 表或视图不存在

2017/05/18 06:17:14 - SPOON4 -

2017/05/18 06:17:14 - SPOON4 -

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.core.database.Database.openQuery(Database.java:1788)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.repository.kdr.delegates.KettleDatabaseRepositoryConnectionDelegate.getIDs(KettleDatabaseRepositoryConnectionDelegate.java:1576)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.repository.kdr.KettleDatabaseRepository.getPartitionSchemaIDs(KettleDatabaseRepository.java:927)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.repository.kdr.delegates.KettleDatabaseRepositoryTransDelegate.readPartitionSchemas(KettleDatabaseRepositoryTransDelegate.java:1021)

2017/05/18 06:17:14 - SPOON4 - ... 36 more

2017/05/18 06:17:14 - SPOON4 - Caused by: java.sql.SQLException: ORA-00942: 表或视图不存在

2017/05/18 06:17:14 - SPOON4 -

2017/05/18 06:17:14 - SPOON4 - at oracle.jdbc.driver.DatabaseError.throwSqlException(DatabaseError.java:112)

2017/05/18 06:17:14 - SPOON4 - at oracle.jdbc.driver.T4CTTIoer.processError(T4CTTIoer.java:331)

2017/05/18 06:17:14 - SPOON4 - at oracle.jdbc.driver.T4CTTIoer.processError(T4CTTIoer.java:288)

2017/05/18 06:17:14 - SPOON4 - at oracle.jdbc.driver.T4C8Oall.receive(T4C8Oall.java:743)

2017/05/18 06:17:14 - SPOON4 - at oracle.jdbc.driver.T4CPreparedStatement.doOall8(T4CPreparedStatement.java:216)

2017/05/18 06:17:14 - SPOON4 - at oracle.jdbc.driver.T4CPreparedStatement.executeForDescribe(T4CPreparedStatement.java:799)

2017/05/18 06:17:14 - SPOON4 - at oracle.jdbc.driver.OracleStatement.executeMaybeDescribe(OracleStatement.java:1037)

2017/05/18 06:17:14 - SPOON4 - at oracle.jdbc.driver.T4CPreparedStatement.executeMaybeDescribe(T4CPreparedStatement.java:839)

2017/05/18 06:17:14 - SPOON4 - at oracle.jdbc.driver.OracleStatement.doExecuteWithTimeout(OracleStatement.java:1132)

2017/05/18 06:17:14 - SPOON4 - at oracle.jdbc.driver.OraclePreparedStatement.executeInternal(OraclePreparedStatement.java:3316)

2017/05/18 06:17:14 - SPOON4 - at oracle.jdbc.driver.OraclePreparedStatement.executeQuery(OraclePreparedStatement.java:3361)

2017/05/18 06:17:14 - SPOON4 - at org.pentaho.di.core.database.Database.openQuery(Database.java:1776)

2017/05/18 06:17:14 - SPOON4 - ... 39 more

2017/05/18 06:17:14 - SPOON4 - ERROR (version 5.2.0.0, build 1 from 2014-09-30_19-48-28 by buildguy) : 一个数据库错误发生在从资源库文件读取转换时

2017/05/18 06:17:14 - SPOON4 - org.pentaho.di.core.exception.KettleDatabaseException:

2017/05/18 06:17:14 - SPOON4 - ERROR executing query

2017/05/18 06:17:14 - SPOON4 - ORA-00942: 表或视图不存在

这是由于oracle11g中，表只有有数据时，才能正式划分空间，否则无法导出，只需要在kettle资源库选择创建或者更新就可以了