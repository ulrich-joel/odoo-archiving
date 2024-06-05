# odoo-archiving
odoo-16

Pour implémenter un module d'archivage compatible avec Odoo 16, nous pouvons adapter le code précédemment proposé aux spécificités d'Odoo 16. Voici un exemple de code pour démarrer avec un module d'archivage dans Odoo 16 :

### 1. Créer le fichier de manifeste (`__manifest__.py`)

```python
{
    'name': 'Document Archive',
    'version': '16.0.1.0.0',
    'summary': 'Module for Document Archiving',
    'author': 'Your Name',
    'category': 'Tools',
    'depends': ['base'],
    'data': [
        'security/ir.model.access.csv',
        'views/document_archive_views.xml',
        'views/document_category_views.xml',
        'views/document_tag_views.xml',
        'views/document_version_views.xml',
        'views/menu_items.xml',
    ],
    'installable': True,
    'application': True,
}
```

### 2. Définir les modèles (`models/document_archive.py`)

```python
from odoo import models, fields, api

class DocumentArchive(models.Model):
    _name = 'document.archive'
    _description = 'Document Archive'

    name = fields.Char(string='Document Name', required=True)
    description = fields.Text(string='Description')
    file = fields.Binary(string='File', attachment=True)
    file_name = fields.Char(string='File Name')
    category_id = fields.Many2one('document.category', string='Category')
    tag_ids = fields.Many2many('document.tag', string='Tags')
    version_ids = fields.One2many('document.version', 'document_id', string='Versions')

class DocumentCategory(models.Model):
    _name = 'document.category'
    _description = 'Document Category'

    name = fields.Char(string='Category Name', required=True)
    document_ids = fields.One2many('document.archive', 'category_id', string='Documents')

class DocumentTag(models.Model):
    _name = 'document.tag'
    _description = 'Document Tag'

    name = fields.Char(string='Tag Name', required=True)

class DocumentVersion(models.Model):
    _name = 'document.version'
    _description = 'Document Version'

    document_id = fields.Many2one('document.archive', string='Document', required=True)
    version_number = fields.Integer(string='Version Number', required=True)
    file = fields.Binary(string='File', attachment=True)
    file_name = fields.Char(string='File Name')
    change_date = fields.Datetime(string='Change Date', default=fields.Datetime.now)
    user_id = fields.Many2one('res.users', string='Changed by', default=lambda self: self.env.user)
```

### 3. Créer les vues (`views/document_archive_views.xml`)

```xml
<odoo>
    <record id="view_document_archive_form" model="ir.ui.view">
        <field name="name">document.archive.form</field>
        <field name="model">document.archive</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <field name="name"/>
                        <field name="description"/>
                        <field name="file"/>
                        <field name="file_name"/>
                        <field name="category_id"/>
                        <field name="tag_ids"/>
                    </group>
                    <notebook>
                        <page string="Versions">
                            <field name="version_ids">
                                <tree editable="bottom">
                                    <field name="version_number"/>
                                    <field name="file"/>
                                    <field name="file_name"/>
                                    <field name="change_date"/>
                                    <field name="user_id"/>
                                </tree>
                            </field>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="view_document_archive_tree" model="ir.ui.view">
        <field name="name">document.archive.tree</field>
        <field name="model">document.archive</field>
        <field name="arch" type="xml">
            <tree>
                <field name="name"/>
                <field name="category_id"/>
                <field name="file_name"/>
                <field name="create_date"/>
            </tree>
        </field>
    </record>

    <record id="view_document_archive_search" model="ir.ui.view">
        <field name="name">document.archive.search</field>
        <field name="model">document.archive</field>
        <field name="arch" type="xml">
            <search>
                <field name="name"/>
                <field name="category_id"/>
                <field name="tag_ids"/>
            </search>
        </field>
    </record>

    <record id="view_document_category_form" model="ir.ui.view">
        <field name="name">document.category.form</field>
        <field name="model">document.category</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <field name="name"/>
                    </group>
                    <notebook>
                        <page string="Documents">
                            <field name="document_ids">
                                <tree>
                                    <field name="name"/>
                                    <field name="file_name"/>
                                </tree>
                            </field>
                        </page>
                    </notebook>
                </sheet>
            </form>
        </field>
    </record>

    <record id="view_document_tag_form" model="ir.ui.view">
        <field name="name">document.tag.form</field>
        <field name="model">document.tag</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <field name="name"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>

    <record id="view_document_version_form" model="ir.ui.view">
        <field name="name">document.version.form</field>
        <field name="model">document.version</field>
        <field name="arch" type="xml">
            <form>
                <sheet>
                    <group>
                        <field name="document_id"/>
                        <field name="version_number"/>
                        <field name="file"/>
                        <field name="file_name"/>
                        <field name="change_date"/>
                        <field name="user_id"/>
                    </group>
                </sheet>
            </form>
        </field>
    </record>
</odoo>
```

### 4. Définir les règles de sécurité (`security/ir.model.access.csv`)

```csv
id,name,model_id:id,group_id:id,perm_read,perm_write,perm_create,perm_unlink
access_document_archive,access_document_archive,model_document_archive,,1,1,1,1
access_document_category,access_document_category,model_document_category,,1,1,1,1
access_document_tag,access_document_tag,model_document_tag,,1,1,1,1
access_document_version,access_document_version,model_document_version,,1,1,1,1
```

### 5. Créer le menu (`views/menu_items.xml`)

```xml
<odoo>
    <menuitem id="menu_document_root" name="Documents" sequence="10"/>

    <menuitem id="menu_document_archive" name="Document Archive" parent="menu_document_root" sequence="10"/>

    <menuitem id="menu_document_category" name="Categories" parent="menu_document_root" sequence="20"/>

    <menuitem id="menu_document_tag" name="Tags" parent="menu_document_root" sequence="30"/>

    <menuitem id="menu_document_version" name="Versions" parent="menu_document_root" sequence="40"/>
</odoo>
```

En suivant ces étapes, vous aurez un module de base pour l'archivage des documents dans Odoo 16. Vous pouvez étendre ces fonctionnalités en fonction des besoins spécifiques de votre organisation.
