Standalone example of JBoss does no longer contain default JMS setup.

To get that, modify standalone.xml or use standalone-full.xml.

These need to be added:

1)
 <extension module="org.jboss.as.messaging"/>

2)

    <socket-binding name="messaging" port="5445"/>

    <socket-binding name="messaging-throughput" port="5455"/>
3)
            <mdb>
                <resource-adapter-ref resource-adapter-name="hornetq-ra"/>
                <bean-instance-pool-ref pool-name="mdb-strict-max-pool"/>
            </mdb>

in

        <subsystem xmlns="urn:jboss:domain:ejb3:1.2">
	
4)

        <subsystem xmlns="urn:jboss:domain:messaging:1.1">
            <hornetq-server>
				<security-enabled>false</security-enabled>
                <persistence-enabled>true</persistence-enabled>
                <journal-file-size>102400</journal-file-size>
                <journal-min-files>2</journal-min-files>

                <connectors>
                    <netty-connector name="netty" socket-binding="messaging"/>
                    <netty-connector name="netty-throughput" socket-binding="messaging-throughput">
                        <param key="batch-delay" value="50"/>
                    </netty-connector>
                    <in-vm-connector name="in-vm" server-id="0"/>
                </connectors>

                <acceptors>
                    <netty-acceptor name="netty" socket-binding="messaging"/>
                    <netty-acceptor name="netty-throughput" socket-binding="messaging-throughput">
                        <param key="batch-delay" value="50"/>
                        <param key="direct-deliver" value="false"/>
                    </netty-acceptor>
                    <in-vm-acceptor name="in-vm" server-id="0"/>
                </acceptors>

                <security-settings>
                    <security-setting match="#">
                        <permission type="send" roles="guest"/>
                        <permission type="consume" roles="guest"/>
                        <permission type="createNonDurableQueue" roles="guest"/>
                        <permission type="deleteNonDurableQueue" roles="guest"/>
                    </security-setting>
                </security-settings>

                <address-settings>
                    <address-setting match="#">
                        <dead-letter-address>jms.queue.DLQ</dead-letter-address>
                        <expiry-address>jms.queue.ExpiryQueue</expiry-address>
                        <redelivery-delay>0</redelivery-delay>
                        <max-size-bytes>10485760</max-size-bytes>
                        <address-full-policy>BLOCK</address-full-policy>
                        <message-counter-history-day-limit>10</message-counter-history-day-limit>
                    </address-setting>
                </address-settings>

                <jms-connection-factories>
                    <connection-factory name="InVmConnectionFactory">
                        <connectors>
                            <connector-ref connector-name="in-vm"/>
                        </connectors>
                        <entries>
                            <entry name="java:/ConnectionFactory"/>
                        </entries>
                    </connection-factory>
                    <connection-factory name="RemoteConnectionFactory">
                        <connectors>
                            <connector-ref connector-name="netty"/>
                        </connectors>
                        <entries>
                            <entry name="RemoteConnectionFactory"/>
                            <entry name="java:jboss/exported/jms/RemoteConnectionFactory"/>
                        </entries>
                    </connection-factory>
                    <pooled-connection-factory name="hornetq-ra">
                        <transaction mode="xa"/>
                        <connectors>
                            <connector-ref connector-name="in-vm"/>
                        </connectors>
                        <entries>
                            <entry name="java:/JmsXA"/>
                        </entries>
                    </pooled-connection-factory>
                </jms-connection-factories>

                <jms-destinations>
                    <jms-queue name="testQueue">
                        <entry name="queue/test"/>
                        <entry name="java:jboss/exported/jms/queue/test"/>
                    </jms-queue>
                    <jms-topic name="testTopic">
                        <entry name="topic/test"/>
                        <entry name="java:jboss/exported/jms/topic/test"/>
                    </jms-topic>
                </jms-destinations>
            </hornetq-server>
        </subsystem>

Note that security has been disabled here. It is enabled by default.		