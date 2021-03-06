.. _mailer:
Mailer module
=============

.. note::
   The Mailer module is not installed by default. It uses phpmailer to send mail.

Installation
------------

In the root of your project:

.. code-block:: bash
   
	composer require phpmv/ubiquity-mailer

Mailer configuration
--------------------

The mailer config file is located in ``app/config/mailer.php``.

Example of configuration for a sending from google mail:

.. code-block:: php
   
   <?php
   return array(
   	"host" => "smtp.gmail.com",
   	"port" => 587,
   	"auth" => true,
   	"user" => "********@gmail.com",
   	"password" => "*******",
   	"protocol" => "smtp",
   	"ns" => "mail",
   	"from" => array(
   		"address" => "***********@gmail.com",
   		"name" => "******"
   	),
   	//Unsecure configuration: only for testing from a local server
   	"SMTPOptions" => array(
   		"ssl" => array(
   			"verify_peer" => false,
   			"verify_peer_name" => false,
   			"allow_self_signed" => true
   		)
   	),
   	"CharSet" => "utf-8"
   );

Creating a Mailer class
-----------------------
A Mailer class is used to prepare the sending of a mail, the elaboration of which can use the same functionalities as the action of a controller.


With the devtools:
******************
.. code-block:: php
   
   Ubiquity newMail InformationMail
   
With the webtools:
******************

In the mailer part:
  - Click on the **Add mailer class** button
  - Enter the folowing values

.. image:: /_static/images/mailer/newMail.png
   :class: bordered

The class is generated by default in the **app/mail** folder (the namespace **mail** is defined by default in the configuration file).

.. code-block:: php
   :caption: app/mail/InformationMail.php
   
   <?php
   namespace mail;
   
   use Ubiquity\mailer\MailerManager;
   
    /**
    * Mailer InformationMail
    **/
   class InformationMail extends \Ubiquity\mailer\AbstractMail {
   
   	/**
   	 *
   	 * {@inheritdoc}
   	 * @see \Ubiquity\mailer\AbstractMail::bodyText()
   	 */
   	public function bodyText() {
   		return 'Message text';
   	}
   
   	/**
   	 *
   	 * {@inheritdoc}
   	 * @see \Ubiquity\mailer\AbstractMail::initialize()
   	 */
   	protected function initialize(){
   		$this->subject = 'Message title';
   		$this->from(MailerManager::loadConfig()['from']??'from@organization');
   		//$this->to($to);
   	}
   
   	/**
   	 *
   	 * {@inheritdoc}
   	 * @see \Ubiquity\mailer\AbstractMail::body()
   	 */
   	public function body() {
   		return '<h1>Message body</h1>';
   	}
   }

Example : An email sent to all users in the database :
******************************************************
.. code-block:: php
   :caption: app/mail/InformationMail.php
   
   namespace mail;
   
   use Ubiquity\mailer\AbstractMail;
   use Ubiquity\orm\DAO;
   use models\User;
   
   class InformationMail extends AbstractMail {
   
   	protected function initialize() {
   		$this->subject = 'Message test';
   		$this->from("myaddressmail@gmail.com", 'jcheron');
   		$this->to(DAO::getAll(User::class, '', false));
   		$this->attachFile('afile.pdf');
   	}
   
   	public function body() {
   		$date = (new \DateTime())->format('c');
   		$user = DAO::getOne(User::class, 1);
   		$body = '<h2>Message</h2><div>Message content</div>';
   		$content = $this->loadView('mailer/AllUsers.html', \compact('date', 'user', 'body'));
   		return $content;
   	}
   
   	public function bodyText() {
   		return 'This message has a text version';
   	}
   }

- ``initialize()`` is automatically invoked during construction and allows to define the attributes of the mail (recipients, subject...).
- ``body()`` and ``bodyText()`` are used to build the body. These methods must return a string, and they can load views.

This message will be visible in the Mailer section of the webtools:

.. image:: /_static/images/mailer/mailerClasses.png
   :class: bordered

